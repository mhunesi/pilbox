Pilbox
======

[![PyPi version](https://badge.fury.io/py/pilbox.svg)](https://pypi.python.org/pypi/pilbox) [![Build Status](https://travis-ci.org/agschwender/pilbox.svg?branch=master)](https://travis-ci.org/agschwender/pilbox) [![Coverage Status](https://coveralls.io/repos/agschwender/pilbox/badge.svg)](https://coveralls.io/r/agschwender/pilbox) [![Code Health](https://landscape.io/github/agschwender/pilbox/master/landscape.svg?style=flat)](https://landscape.io/github/agschwender/pilbox/master)

Pilbox is an image processing application server built on Python's [Tornado web framework](http://www.tornadoweb.org/en/stable/) using the [Python Imaging Library (Pillow)](https://pypi.python.org/pypi/Pillow/). It is not intended to be the primary source of images, but instead acts as a proxy which requests images and resizes them as desired.

Setup
=====

Dependencies
------------

  * [Python >= 2.7](http://www.python.org/download/)
  * [Pillow 2.9.0](https://pypi.python.org/pypi/Pillow/2.9.0)
  * [Tornado 4.5.1](https://pypi.python.org/pypi/tornado/4.5.1)
  * [OpenCV 2.x](http://opencv.org/) (optional)
  * [PycURL 7.x](http://pycurl.sourceforge.net/) (optional, but recommended; required for proxy requests and requests over TLS)
  * Image Libraries: libjpeg-dev, libfreetype6-dev, libwebp-dev, zlib1g-dev, liblcms2-dev

Pilbox highly recommends installing `libcurl` and `pycurl` in order to get better HTTP request performance as well as additional features such as proxy requests and requests over TLS. Installed versions of `libcurl` should be a minimum of `7.21.1` and `pycurl` should be a minimum of `7.18.2`. Furthermore, it is recommended that the `libcurl` installation be built with asynchronous DNS resolver (threaded or c-ares), otherwise it may encounter various problems with request timeouts (for more information, see [CURLOPT_CONNECTTIMEOUT_MS](http://curl.haxx.se/libcurl/c/curl_easy_setopt.html#CURLOPTCONNECTTIMEOUTMS) and comments in [curl_httpclient.py](https://github.com/tornadoweb/tornado/blob/master/tornado/curl_httpclient.py)

Install
-------

Pilbox can be installed with pip

    $ pip install pilbox

Or easy_install

    $ easy_install pilbox

Or from source

    $ git clone https://github.com/agschwender/pilbox.git

Packaged with Pilbox is a [Vagrant](http://www.vagrantup.com/) configuration file which installs all necessary dependencies on a virtual box using [Ansible](http://www.ansibleworks.com/). See the [Vagrant documentation](http://docs.vagrantup.com/v2/installation/) and the [Ansible documentation](http://www.ansibleworks.com/docs/gettingstarted.html#getting-ansible) for installation instructions. Once installed, the following will start and provision a virtual machine.

    $ vagrant up
    $ vagrant provision

To access the virtual machine itself, simply...

    $ vagrant ssh

When running via Vagrant, the application is automatically started on port 8888 on 192.168.100.100, i.e.

    http://192.168.100.100:8888/

Running
=======

To run the application, issue the following command

    $ python -m pilbox.app

By default, this will run the application on port 8888 and can be accessed by visiting:

    http://localhost:8888/

To see a list of all available options, run

    $ python -m pilbox.app --help
    Usage: pilbox/app.py [OPTIONS]

    Options:

      --allowed_hosts            list of allowed hosts (default [])
      --allowed_operations       list of allowed operations (default [])
      --background               default hexadecimal bg color (RGB or ARGB)
      --client_key               client key
      --client_name              client name
      --config                   path to configuration file
      --content_type_from_image  override content type using image mime type
      --debug                    run in debug mode (default False)
      --expand                   default to expand when rotating
      --filter                   default filter to use when resizing
      --help                     show this help information
      --implicit_base_url        prepend protocol/host to url paths
      --max_operations           maximum operations to perform (default 10)
      --max_requests             max concurrent requests (default 40)
      --max_resize_height        maximum resize height (default 15000)
      --max_resize_width         maximum resize width (default 15000)
      --operation                default operation to perform
      --optimize                 default to optimize when saving
      --port                     run on the given port (default 8888)
      --position                 default cropping position
      --progressive              default to progressive when saving
      --proxy_host               proxy hostname
      --proxy_port               proxy port
      --quality                  default jpeg quality, 0-100 or keep
      --timeout                  timeout of requests in seconds (default 10)
      --user_agent               user agent
      --validate_cert            validate certificates (default True)

Calling
=======

To use the image processing service, include the application url as you would any other image. E.g. this image url

```html
<img src="http://i.imgur.com/zZ8XmBA.jpg" />
```

Would be replaced with this image url

```html
<img src="http://localhost:8888/?url=http%3A%2F%2Fi.imgur.com%2FzZ8XmBA.jpg&w=300&h=300&mode=crop" />
```

This will request the image served at the supplied url and resize it to `300x300` using the `crop` mode. The below is the list of parameters that can be supplied to the service.

General Parameters
------------------

  * _url_: The url of the image to be resized
  * _op_: The operation to perform: noop, resize (default), rotate
    * _noop_: No operation is performed, image is returned as it is received
    * _region_: Select a sub-region from the image
    * _resize_: Resize the image
    * _rotate_: Rotate the image
  * _fmt_: The output format to save as, defaults to the source format
    * _gif_: Save as GIF
    * _jpeg_: Save as JPEG
    * _png_: Save as PNG
    * _tiff_: Save as TIFF
    * _webp_: Save as WebP
  * _bg_: Background color used with images that have transparency; useful when saving to a format that does not support transparency
    * _RGB_: 3- or 6-digit hexadecimal number
    * _ARGB_: 4- or 8-digit hexadecimal number, only relevant for PNG images
  * _opt_: The output should be optimized, only relevant to JPEGs and PNGs
  * _exif_: Keep original [Exif](http://en.wikipedia.org/wiki/Exchangeable_image_file_format) data in the processed image, only relevant for JPEG
  * _prog_: Enable progressive output, only relevant to JPEGs
  * _q_: The quality, (1-99) or keep, used to save the image, only relevant to JPEGs


Resize Parameters
-----------------

  * _w_: The desired width of the image
  * _h_: The desired height of the image
  * _mode_: The resizing method: adapt, clip, crop (default), fill and scale
    * _adapt_: Resize using crop if the resized image retains a supplied percentage of the original image; otherwise fill
    * _clip_: Resize to fit within the desired region, keeping aspect ratio
    * _crop_: Resize so one dimension fits within region, center, cut remaining
    * _fill_: Fills the clipped space with a background color
    * _scale_: Resize to fit within the desired region, ignoring aspect ratio
  * _bg_: Background color used with fill mode (RGB or ARGB)
    * _RGB_: 3- or 6-digit hexadecimal number
    * _ARGB_: 4- or 8-digit hexadecimal number, only relevant for PNG images
  * _filter_: The filtering algorithm used for resizing
    * _nearest_: Fastest, but often images appear pixelated
    * _bilinear_: Faster, can produce acceptable results
    * _bicubic_: Fast, can produce acceptable results
    * _antialias_: Slower, produces the best results
  * _pos_: The crop position
    * _top-left_: Crop from the top left
    * _top_: Crop from the top center
    * _top-right_: Crop from the top right
    * _left_: Crop from the center left
    * _center_: Crop from the center
    * _right_: Crop from the center right
    * _bottom-left_: Crop from the bottom left
    * _bottom_: Crop from the bottom center
    * _bottom-right_: Crop from the bottom right
    * _face_: Identify faces and crop from the midpoint of their position(s)
    * _x,y_: Custom center point position ratio, e.g. 0.0,0.75
  * _retain_: The minimum percentage (1-99) of the original image that must still be visible in the resized image in order to use crop mode

Region Parameters
-----------------

  * _rect_: The region as x,y,w,h; x,y: top-left position, w,h: width/height of region

Rotate Parameters
-----------------

  * _deg_: The desired rotation angle degrees
    * 0-359: The number of degrees to rotate (clockwise)
    * auto: Auto rotation based on EXIF orientation, only relevant to JPEGs
  * _expand_: Expand the size to include the full rotated image

Security-related Parameters
---------------------------

  * _client_: The client name
  * _sig_: The signature

The `url` parameter is always required as it dictates the image that will be manipulated. `op` is optional and defaults to `resize`. It also supports a comma separated list of operations, where each operation is applied in the order that it appears in the list. Depending on the operation, additional parameters are required.  All image manipulation requests accept `fmt`, `opt` and `q`. `fmt` is optional and defaults to the source image format. `opt` is optional and defaults to `0`.  `q` is optional and defaults to `90`. To ensure security, all requests also support, `client` and `sig`. `client` is required only if the `client_name` is defined within the configuration file. Likewise, `sig` is required only if the `client_key` is defined within the configuration file. See the [signing section](#signing) for details on how to generate the signature.

For resizing, either the `w` or `h` parameter is required. If only one dimension is specified, the application will determine the other dimension using the aspect ratio. `mode` is optional and defaults to `crop`. `filter` is optional and defaults to `antialias`.  `bg` is optional and defaults to `fff`. `pos` is optional and defaults to `center`. `retain` is optional and defaults to `75`.

For region sub-selection, `rect` is required. For rotating, `deg` is required. `expand` is optional and defaults to `0` (disabled). It is recommended that this feature not be used as it typically does not produce high quality images.

Note, all built-in defaults can be overridden by setting them in the configuration file. See the [configuration section](#configuration) for more details.

Examples
========

The following images show the various resizing modes in action for an original image size of `640x428` that is being resized to `500x400`.

Clip
----

The image is resized to fit within a `500x400` box, maintaining aspect ratio and producing an image that is `500x334`. Clipping is useful when no portion of the image can be lost and it is acceptable that the image not be exactly the supplied dimensions, but merely fit within the dimensions.

![Clipped image](https://github.com/agschwender/pilbox/raw/master/pilbox/test/data/expected/example-500x400-mode=clip.jpg)

Crop
----

The image is resized so that one dimension fits within the `500x400` box. It is then centered and the excess is cut from the image. Cropping is useful when the position of the subject is known and the image must be exactly the supplied size.

![Cropped image](https://github.com/agschwender/pilbox/raw/master/pilbox/test/data/expected/example-500x400-mode=crop.jpg)

Fill
----

Similar to clip, fill resizes the image to fit within a `500x400` box. Once clipped, the image is centered within the box and all left over space is filled with the supplied background color. Filling is useful when no portion of the image can be lost and it must be exactly the supplied size.

![Filled image](https://github.com/agschwender/pilbox/raw/master/pilbox/test/data/expected/example-500x400-mode=fill-background=ccc.jpg)

Scale
-----

The image is clipped to fit within the `500x400` box and then stretched to fill the excess space. Scaling is often not useful in production environments as it generally produces poor quality images. This mode is largely included for completeness.

![Scale image](https://github.com/agschwender/pilbox/raw/master/pilbox/test/data/expected/example-500x400-mode=scale.jpg)


Testing
=======

To run all tests, issue the following command

    $ python -m pilbox.test.runtests

To run individual tests, simply indicate the test to be run, e.g.

    $ python -m pilbox.test.runtests pilbox.test.signature_test

Signing
=======

In order to secure requests so that unknown third parties cannot easily use the resize service, the application can require that requests provide a signature. To enable this feature, set the `client_key` option. The signature is a hexadecimal digest generated from the client key and the query string using the HMAC-SHA1 message authentication code (MAC) algorithm. The below python code provides an example implementation.

```python
import hashlib
import hmac

def derive_signature(key, qs):
    m = hmac.new(key, None, hashlib.sha1)
    m.update(qs)
    return m.hexdigest()
```

The signature is passed to the application by appending the `sig` parameter to the query string; e.g. `x=1&y=2&z=3&sig=c9516346abf62876b6345817dba2f9a0c797ef26`. Note, the application does not include the leading question mark when verifying the supplied signature. To verify your signature implementation, see the `pilbox.signature` command described in the [tools section](#tools).

Configuration
=============

All options that can be supplied to the application via the command line, can also be specified in the configuration file. Configuration files are simply python files that define the options as variables. The below is an example configuration.

    # General settings
    port = 8888

    # Set client name and key if the application requires signed requests. The
    # client must sign the request using the client_key, see README for
    # instructions.
    client_name = "sample"
    client_key = "3NdajqH8mBLokepU4I2Bh6KK84GUf1lzjnuTdskY"

    # Set the allowed hosts as an alternative to signed requests. Only those
    # images which are served from the following hosts will be requested.
    allowed_hosts = ["localhost"]

    # Request-related settings
    max_requests = 50
    timeout = 7.5

    # Set default resizing options
    background = "ccc"
    filter = "bilinear"
    mode = "crop"
    position = "top"

    # Set default rotating options
    expand = False

    # Set default saving options
    format = None
    optimize = 1
    quality = 90

Tools
=====

To verify that your client application is generating correct signatures, use the signature command.

    $ python -m pilbox.signature --key=abcdef "x=1&y=2&z=3"
    Query String: x=1&y=2&z=3
    Signature: c9516346abf62876b6345817dba2f9a0c797ef26
    Signed Query String: x=1&y=2&z=3&sig=c9516346abf62876b6345817dba2f9a0c797ef26

The application allows the use of the resize functionality via the command line.

    $ python -m pilbox.image --width=300 --height=300 http://i.imgur.com/zZ8XmBA.jpg > /tmp/foo.jpg

If a new mode is added or a modification was made to the libraries that would change the current expected output for tests, run the generate test command to regenerate the expected output for the test cases.

    $ python -m pilbox.test.genexpected

Deploying
=========

The application itself does not include any caching. It is recommended that the application run behind a CDN for larger applications or behind varnish for smaller ones.

Defaults for the application have been optimized for quality rather than performance. If you wish to get higher performance out of the application, it is recommended you use a less computationally expensive filtering algorithm and a lower JPEG quality. For example, add the following to the configuration.

    # Set default resizing options
    filter = "bicubic"
    quality = 75

If you wish to improve performance further and are using an x86 platform, you may want to consider using [Pillow-SIMD]( https://github.com/uploadcare/pillow-simd/). Follow the steps in [Installation](https://github.com/uploadcare/pillow-simd#installation) and it should function as a drop-in replacement for `Pillow`. To avoid any incompatibility issues, use the same version of `Pillow-SIMD` as is being used for `Pillow`.

Extension
=========

While it is generally recommended to use Pilbox as a standalone server, it can also be used as a library. To extend from it and build a custom image processing server, use the following example.

    #!/usr/bin/env python

    import tornado.gen

    from pilbox.app import PilboxApplication, ImageHandler, main


    class CustomApplication(PilboxApplication):
        def get_handlers(self):
            return [(r"/(\d+)x(\d+)/(.+)", CustomImageHandler)]


    class CustomImageHandler(ImageHandler):
        def prepare(self):
            self.args = self.request.arguments.copy()

        @tornado.gen.coroutine
        def get(self, w, h, url):
            self.args.update(dict(w=w, h=h, url=url))

            self.validate_request()
            resp = yield self.fetch_image()
            self.render_image(resp)

        def get_argument(self, name, default=None):
            return self.args.get(name, default)


    if __name__ == "__main__":
        main(app=CustomApplication())

Changelog
=========

  * 0.1: Image resizing fit
  * 0.1.1: Image cropping
  * 0.1.2: Image scaling
  * 0.2: Configuration integration
  * 0.3: Signature generation
  * 0.3.1: Signature command-line tool
  * 0.4: Image resize command-line tool
  * 0.5: Facial recognition cropping
  * 0.6: Fill resizing mode
  * 0.7: Resize using crop position
  * 0.7.1: Resize using a single dimension, maintaining aspect ratio
  * 0.7.2: Added filter and quality options
  * 0.7.3: Support python 3
  * 0.7.4: Fixed cli for image generation
  * 0.7.5: Write output in 16K blocks
  * 0.8: Added support for ARGB (alpha-channel)
  * 0.8.1: Increased max clients and write block sizes
  * 0.8.2: Added configuration for max clients and timeout
  * 0.8.3: Only allow http and https protocols
  * 0.8.4: Added support for WebP
  * 0.8.5: Added format option and configuration overrides for mode and format
  * 0.8.6: Added custom position support
  * 0.9: Added rotate operation
  * 0.9.1: Added sub-region selection operation
  * 0.9.4: Added Pilbox as a PyPI package
  * 0.9.10: Converted README to reStructuredText
  * 0.9.14: Added Sphinx docs
  * 0.9.15: Added implicit base url to configuration
  * 0.9.16: Added validate cert to configuration
  * 0.9.17: Added support for GIF format
  * 0.9.18: Fix for travis builds on python 2.6 and 3.3
  * 0.9.19: Validate cert fix
  * 0.9.20: Added optimize option
  * 0.9.21: Added console script entry point
  * 1.0.0: Modified for easier library usage
  * 1.0.1: Added allowed operations and default operation
  * 1.0.2: Modified to allow override of http content type
  * 1.0.3: Safely catch image save errors
  * 1.0.4: Added progressive option
  * 1.1.0: Proxy server support
  * 1.1.1: Added JPEG auto rotation based on EXIF orientation
  * 1.1.2: Added keep JPEG quality option and set JPEG subsampling to keep
  * 1.1.3: Fixed auto rotation on JPEG with missing EXIF data
  * 1.1.4: Exception handling around invalid EXIF data
  * 1.1.5: Fixed image requests without content types
  * 1.1.6: Support custom applications that need command line arguments
  * 1.1.7: Support adapt resize mode
  * 1.1.8: Added preserve Exif flag
  * 1.1.9: Increased Pillow version to 2.8.1
  * 1.1.10: Added ca_certs option
  * 1.1.11: Added support for TIFF
  * 1.2.0: Support setting background when saving a transparent image
    * _Backwards incompatible_: default background property changed to `0fff`. To restore previous behavior, set background in config to `ffff`.
  * 1.2.1: Added max operations config property
  * 1.2.2: Added max resize width and height config properties
  * 1.2.3: Added user_agent option
  * 1.3.0: Increased Pillow to 2.9.0 and Tornado to 4.5.1