---
layout: post
title:  "reicast, websockets & opengl (es) streaming"
categories: jekyll update
img: reicast-websocket-gl.gif
---


image

For some reason, I was always fascinated with the idea of dreamcast on the cloud. The initial idea was to use amazon g2 instances + the grid api, but it felt like too much work for today, and my debit card was empty too. 

Since reicast already has a websocket server build in (for some future debugger/profiler UI), using websockets seemed like a great idea.

So....

Idea 1 - glreadpixels + websocket + libjpeg turbo + img tag

This one failed pretty fast. libjpeg turbo is quite big and didn't seem very cooperative on getting included under my makefile. Plus, I had to make it work with Visual Studio, and meh. Didn't get much further than fetching and extracting the source tarball.

Idea 2 - glreadpixels + websocket + bmp + img tag

Right. Streaming over localhost -for now- so bmp it is. Some quick googling directed me to this fine post which had clean, working bmp generation code.

Andd the magic

gles.cpp, global scope

    //yes, globals are evil for your soul.

    int bmp_filesize;

    u8* bmp_pdata;

in gl_swap(), before eglSwapBuffers

    {

    int w = screen_width;
    int h = screen_height;

    int filesize = bmp_filesize = 54 + 3*w*h; //w is your image width, h is image height, both int

    if (!bmp_pdata)
    bmp_pdata = new u8[14 + 40 + filesize];

    u8* img = bmp_pdata + 14 + 40;
    glReadPixels(0,0,screen_width,screen_height,GL_RGB, GL_UNSIGNED_BYTE,img);

    unsigned char bmpfileheader[14] = {'B','M', 0,0,0,0, 0,0, 0,0, 54,0,0,0};
    unsigned char bmpinfoheader[40] = {40,0,0,0, 0,0,0,0, 0,0,0,0, 1,0, 24,0};
    unsigned char bmppad[3] = {0,0,0};

    bmpfileheader[ 2] = (unsigned char)(filesize );
    bmpfileheader[ 3] = (unsigned char)(filesize>> 8);
    bmpfileheader[ 4] = (unsigned char)(filesize>>16);
    bmpfileheader[ 5] = (unsigned char)(filesize>>24);

    bmpinfoheader[ 4] = (unsigned char)( w );
    bmpinfoheader[ 5] = (unsigned char)( w>> 8);
    bmpinfoheader[ 6] = (unsigned char)( w>>16);
    bmpinfoheader[ 7] = (unsigned char)( w>>24);
    bmpinfoheader[ 8] = (unsigned char)( h );
    bmpinfoheader[ 9] = (unsigned char)( h>> 8);
    bmpinfoheader[10] = (unsigned char)( h>>16);
    bmpinfoheader[11] = (unsigned char)( h>>24);

    memcpy(bmp_pdata+0,bmpfileheader,14);
    memcpy(bmp_pdata+14,bmpinfoheader,40);

    }

in server.cpp, global scope

    //Including externs this way is bad. Don't do it or your project will end up like reicast. Use header files, kthx.

    extern int bmp_filesize;
    extern u8* bmp_pdata;

and in callback_dumb_increment, case LWS_CALLBACK_SERVER_WRITEABLE, replace the write with

    m = libwebsocket_write(wsi, bmp_pdata, bmp_filesize, LWS_WRITE_BINARY);

    if (m < bmp_filesize) {
    lwsl_err("ERROR %d writing to di socket\n", n);
    return -1;
    }

And in debugger.html, somewhere in <body>

    <img id="dyn_img" />

as well as function got_packet

    socket_di.onmessage =function got_packet(msg) {
    //document.getElementById("number").textContent = msg.data + "\n";
    var img = document.getElementById("dyn_img");
    var old = img.src;
    img.src = URL.createObjectURL(msg.data);
    URL.revokeObjectURL(old);
    }

Building, booting the bios, annddd
image

Woohoo, it works! It stopped after 50 frames or so though. Then I realised, in server.cpp, some stuff had to be /**/

    /*

    //Yeah, let's keep the connection open, alrighty?

    if (close_testing && pss->number == 50) {
    lwsl_info("close tesing limit, closing\n");
    return -1;
    }
    */

Good. This resulted in reicast crashing as soon as complex bitmaps got rendered. Some quick stacktracing lead to the conclusion that deflated frames are to blame.

extention_deflate_frame.c, lws_extension_callback_deflate_frame(), case LWS_EXT_CALLBACK_PAYLOAD_TX, 

    if (conn->buf_out_length > LWS_MAX_ZLIB_CONN_BUFFER) {
    lwsl_ext("zlib out hit limit %u\n",
    LWS_MAX_ZLIB_CONN_BUFFER);
    return -1;
    }

and, private-websockets.h, 

    #ifndef LWS_MAX_ZLIB_CONN_BUFFER
    #define LWS_MAX_ZLIB_CONN_BUFFER (64 * 1024)
    #endif

Complex images get deflated to more than 64K, and that was the problem. I thought of changing the constant, but fighting with the lib felt complicated, and what if it's part of the spec? And why, really, 64K and not some other power of 2? Argghh.

So, instead, in extention.c I replaced the #endif with #elif 0, disabling compression.

    struct libwebsocket_extension libwebsocket_internal_extensions[] = {
    #ifdef LWS_EXT_DEFLATE_STREAM
    {
    "deflate-stream",
    lws_extension_callback_deflate_stream,
    sizeof(struct lws_ext_deflate_stream_conn)
    },
    #elif 0
    {
    "x-webkit-deflate-frame",
    lws_extension_callback_deflate_frame,
    sizeof(struct lws_ext_deflate_frame_conn)
    },
    {
    "deflate-frame",
    lws_extension_callback_deflate_frame,
    sizeof(struct lws_ext_deflate_frame_conn)
    },
    #endif //yes, I also added a proper endif here. Kudos for reading the code blocks!
    { /* terminator */
    NULL, NULL, 0
    }
    };

Yay. Dreamacst bios menu! The swapped colors are to be expected (bmp/rgb vs ogl/bgr), but otherwise it worked. For a minute.

Then,
image

Hum, mnnn. So much for URI.revokeObjectURL. Or maybe the images leak. Who knows. Some googling was enough to convince me this is a fairly common issue with blog urls and images. Another dead end it seems. And it did work nicely, otherwise.

Idea 3 - glreadpixels + websocket + webgl

Thinking of another way to render the images, webgl poped into mind. It must have glTexture2D. It must support arraybuffers. Right? And .bmps are just bitmaps if you skip the headers .

Again, google pointed me to what I was looking for, namely this nice webgl demo from mozilla.

in webgl-demo.js, initTextures

    //don't load stuff here, just create the id, thanks.

    function initTextures() {
    cubeTexture = gl.createTexture();
    }

And somewhere above that in global scope

    var socket_di;

    if (typeof MozWebSocket != "undefined") {
    socket_di = new MozWebSocket("ws://127.0.0.1:5678",
    "dumb-increment-protocol");
    } else {
    socket_di = new WebSocket("ws://127.0.0.1:5678",
    "dumb-increment-protocol");
    }

    socket_di.binaryType = "arraybuffer";

    socket_di.onmessage =function got_packet(msg) {

    gl.texImage2D(
    gl.TEXTURE_2D, // target
    0, // mip level
    gl.RGB, // internal format
    512, 512, // width and height
    0, // border
    gl.RGB, //format
    gl.UNSIGNED_BYTE, // type
    new Uint8Array(msg.data,54) // texture data
    );
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_NEAREST);
    gl.generateMipmap(gl.TEXTURE_2D);
    gl.bindTexture(gl.TEXTURE_2D, null);
    }

Annddd wooosshh!
image

Works. A bit glitchy as the sent bitmap has wrong dimentions.

Thus, in winmain.cpp

    // Width and height of the window

    #define WINDOW_WIDTH 512
    #define WINDOW_HEIGHT 512

Fixed it
image

Yay, rotating dreamcast uh cube uh over localhost. Must be useful for something. Maybe for that pvr 3d debugger idea..

I'll cleanup the code, make it properly handle NPOTs, get rid of the rotation and it should end up in the reicast git somewhere next week.