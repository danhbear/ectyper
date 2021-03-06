Ectyper
==========

Set of Tornado request handlers that allows on-the-fly conversion of images
according to a request's query string options.  Ectyper, by default, provides
a way to resize, reflect and reformat images.

Synopsis
==========

    from ectyper.handlers import ImageHandler
    import os
    from tornado import web, ioloop
  
    class StreamLocal(ImageHandler):
        """
        Trivial local file handler.  It loads files in images directory, converts
        it and streams it to the client.
    
        Example calls:      
          Resize:
          http://host:8888/images/hulu.jpg?size=200x96
          
          Reformat:
          http://host:8888/images/hulu.jpg?format=png
          
          Reflect:
          http://host:8888/images/hulu.jpg?size=200x96&format=png&reflection_height=60
        """
        
        def calculate_options(self):
            super(StreamLocal, self).calculate_options()
  
            # By default Ectyper uses "convert" as found in the PATH of the running
            # python process, use this to override.
            if self.magick:
                self.magick.convert_path = "/path/to/imagemagick/convert"
  
        def handler(self, *args):
            self.convert_image(os.path.join("/path/to/images", args[0]))
  
    app = web.Application([
        ('/images/(.*)', StreamLocal),
    ])
  
    if __name__ == "__main__":
        app.listen(8888)
        ioloop.IOLoop.instance().start()

Description
==========

Ectyper provides a primary ImageHandler class (extending Tornado's RequestHandler
class).  This class parses each request's query string and converts it into a
command-line call into ImageMagick's convert.  This class is wrapped by
ectyper.magick.ImageMagick which can be overridden to provide other options.

A request into an ImageHandler will automatically generate an ImageMagick
object based on standard options (see help(ectyper.handlers.ImageHandler) for
more).  Once your handler calls convert_image() with a local file path or a
remote URL, ImageHandler kicks off an ImageMagick convert process with the
calculated options and streams the result to the browser.  You can optionally
extend CachingImageHandler or FileCachingImageHandler to stream the output
elsewhere for caching purposes.

Full list of options supported by default:

    size=NxM
       Resize the source image to N pixels wide and M pixels high.
 
    maintain_ratio=1
       Maintain aspect ratio when resizing (ignored if size is not
       provided).  If requested size is a different aspect ratio than
       the source image, the resize will scale to fit the width or height,
       whichever is reached first.  The other dimension will be centered
       vertically or horizontally as necessary.  Defaults to 0.
 
    reflection_height=N
       Flip the image upside down and apply a gradient to mimic a reflected
       image.  reflection_alpha_top and reflection_alpha_bottom can be used to
       set the gradient parameters.
 
    reflection_alpha_top=N
       The top value to use when generating the gradient for reflection_height,
       ignored if that parameter is not set.  Should be between 0 and 1.
       Defaults to 1.  Note that reflection alpha only works properly with
       format=png output.
 
    reflection_alpha_bottom=N
       The bottom value to use when generating the gradient for
       reflection_height, ignored if that parameter is not set.  Should be
       between 0 and 1.  Defaults to 0.
 
    format=(jpeg|png|png16)
       Format to convert the image into.  png16 is 24-bit png pre-dithered
       for 16-bit (RGB555) screens.  Defaults to jpeg.

Examples
==========

See example.py for more.

License
==========

Copyright (C) 2010-2011 by Hulu, LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

Contact
==========

For support or for further questions, please contact:

      ectyper-dev@googlegroups.com
