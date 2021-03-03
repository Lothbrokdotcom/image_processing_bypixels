# Image Processing By Pixels using C#
Fork of code at https://www.codeproject.com/Articles/33838/Image-Processing-using-C
## Application
The application uses the basic Windows Forms application. I have handled the images with a separate class called ImageHandler in which all the image related operations are done including Saving, Graphics related operations. The Functionality includes getting image information, zooming, color filtering, brightening, contrasting, gamma filtering, grayscale filtering, invert filtering, resizing with full resolution, rotating and flipping, cropping and inserting text, any other image and some geometric shapes. Scrolling is achieved in the standard manner. The Paint method uses the AutoScrollPosition property to find out our scroll position, which is set by using the AutoScrollMinSize property.

### 1. Color Filter
Color filters are sometimes classified according to their type of spectral absorption: short-wavelength pass, long-wavelength pass or band-pass; diffuse or sharp-cutting; monochromatic or conversion. The short-wavelength pass transmits all wavelengths up to the specified one and then absorbs. The long-wavelength pass is the opposite. Every filter is a band-pass filter when considered generally.

It is very simple - it just adds or subtracts a value to each color. The most useful thing to do with this filter is to set two colors to -255 in order to strip them and see one color component of an image. For example, for red filter, keep the red component as it is and just subtract 255 from the green component and blue component.

      public void SetColorFilter(ColorFilterTypes colorFilterType)
      {
        Bitmap temp = (Bitmap)_currentBitmap;
        Bitmap bmap = (Bitmap)temp.Clone();
        Color c;
        for (int i = 0; i < bmap.Width; i++)
        {
          for (int j = 0; j < bmap.Height; j++)
          {
            c = bmap.GetPixel(i, j);
            int nPixelR = 0;
            int nPixelG = 0;
            int nPixelB = 0;
            if (colorFilterType == ColorFilterTypes.Red)
            {
              nPixelR = c.R;
              nPixelG = c.G - 255;
              nPixelB = c.B - 255;
            }
            else if (colorFilterType == ColorFilterTypes.Green)
            {
              nPixelR = c.R - 255;
              nPixelG = c.G;
              nPixelB = c.B - 255;
            }
            else if (colorFilterType == ColorFilterTypes.Blue)
            {
              nPixelR = c.R - 255;
              nPixelG = c.G - 255;
              nPixelB = c.B;
            }
            nPixelR = Math.Max(nPixelR, 0);
            nPixelR = Math.Min(255, nPixelR);

            nPixelG = Math.Max(nPixelG, 0);
            nPixelG = Math.Min(255, nPixelG); 

            nPixelB = Math.Max(nPixelB, 0);
            nPixelB = Math.Min(255, nPixelB);

            bmap.SetPixel(i, j, Color.FromArgb((byte)nPixelR,
            (byte)nPixelG, (byte)nPixelB));
          }
        }
        _currentBitmap = (Bitmap)bmap.Clone();
      }
### 2. Gamma
Gamma filtering matters if you have any interest in displaying an image accurately on a computer screen. Gamma filtering controls the overall brightness of an image. Images which are not properly corrected can look either bleached out, or too dark. Trying to reproduce colors accurately also requires some knowledge of gamma. Varying the amount of gamma filtering changes not only the brightness, but also the ratios of red to green to blue. We produce a new color array and take the colors from that as the respective components in the image. The input values range between 0.2 to 5.
    
        public void SetGamma(double red, double green, double blue)
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          Color c;
          byte[] redGamma = CreateGammaArray(red);
          byte[] greenGamma = CreateGammaArray(green);
          byte[] blueGamma = CreateGammaArray(blue);
          for (int i = 0; i < bmap.Width; i++)
          {
            for (int j = 0; j < bmap.Height; j++)
            {
              c = bmap.GetPixel(i, j);
              bmap.SetPixel(i, j, Color.FromArgb(redGamma[c.R],
              greenGamma[c.G], blueGamma[c.B]));
            }
          }
          _currentBitmap = (Bitmap)bmap.Clone();
        }
    
Gamma array is created as:

      private byte[] CreateGammaArray(double color)
      {
        byte[] gammaArray = new byte[256];
        for (int i = 0; i < 256; ++i)
        {
          gammaArray[i] = (byte)Math.Min(255,
          (int)((255.0 * Math.Pow(i / 255.0, 1.0 / color)) + 0.5));
        }
        return gammaArray;
      }
### 3. Brightness
Brightening images are sometimes needed, it's a personal choice. Sometimes printing needs a lighter image than viewing. It is done just by adjusting the color components as per the user requirement. The input ranges between -255 and 255.

        public void SetBrightness(int brightness)
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          if (brightness < -255) brightness = -255;
          if (brightness > 255) brightness = 255;
          Color c;
          for (int i = 0; i < bmap.Width; i++)
          {
            for (int j = 0; j < bmap.Height; j++)
            {
              c = bmap.GetPixel(i, j);
              int cR = c.R + brightness;
              int cG = c.G + brightness;
              int cB = c.B + brightness;

              if (cR < 0) cR = 1;
              if (cR > 255) cR = 255;

              if (cG < 0) cG = 1;
              if (cG > 255) cG = 255;

              if (cB < 0) cB = 1;
              if (cB > 255) cB = 255;

              bmap.SetPixel(i, j,
              Color.FromArgb((byte)cR, (byte)cG, (byte)cB));
            }
          }
          _currentBitmap = (Bitmap)bmap.Clone();
        }
### 4. Contrast
Contrasting of images is certainly a complex processing. Instead of just moving all the pixels in the particular direction, we must either increase or decrease the difference between the set of pixels. We accept values between -100 and 100, but we turn these into a double between the values of 0 and 4.

          public void SetContrast(double contrast)
          {
            Bitmap temp = (Bitmap)_currentBitmap;
            Bitmap bmap = (Bitmap)temp.Clone();
            if (contrast < -100) contrast = -100;
            if (contrast > 100) contrast = 100;
            contrast = (100.0 + contrast) / 100.0;
            contrast *= contrast;
            Color c;
            for (int i = 0; i < bmap.Width; i++)
            {
              for (int j = 0; j < bmap.Height; j++)
              {
                c = bmap.GetPixel(i, j);
                double pR = c.R / 255.0;
                pR -= 0.5;
                pR *= contrast;
                pR += 0.5;
                pR *= 255;
                if (pR < 0) pR = 0;
                if (pR > 255) pR = 255;

                double pG = c.G / 255.0;
                pG -= 0.5;
                pG *= contrast;
                pG += 0.5;
                pG *= 255;
                if (pG < 0) pG = 0;
                if (pG > 255) pG = 255;

                double pB = c.B / 255.0;
                pB -= 0.5;
                pB *= contrast;
                pB += 0.5;
                pB *= 255;
                if (pB < 0) pB = 0;
                if (pB > 255) pB = 255;

                bmap.SetPixel(i, j,
                Color.FromArgb((byte)pR, (byte)pG, (byte)pB));
              }
            }
            _currentBitmap = (Bitmap)bmap.Clone();
          }
### 5. Grayscale
Gray scale filtering is in reference to the color mode of a particular image.A gray scale image would, in layman's terms, be a black and white image, any other color would not be included in it.

Basically, it's a black and white image, the colors in that image, if any will be converted to corresponding shade of gray (mid tones between black and white) thus, making each bit of the image still differentiable.

        public void SetGrayscale()
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          Color c;
          for (int i = 0; i < bmap.Width; i++)
          {
            for (int j = 0; j < bmap.Height; j++)
            {
              c = bmap.GetPixel(i, j);
              byte gray = (byte)(.299 * c.R + .587 * c.G + .114 * c.B);

              bmap.SetPixel(i, j, Color.FromArgb(gray, gray, gray));
            }
          }
          _currentBitmap = (Bitmap)bmap.Clone();
        }
### 6. Invert
This is so simple that it doesn't even matter that the color components are out of order. it is just taking the opposite color of the current component. that is for example if the color component is 00 then the opposite we get is FF (255-0).

        public void SetInvert()
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          Color c;
          for (int i = 0; i < bmap.Width; i++)
          {
            for (int j = 0; j < bmap.Height; j++)
            {
              c = bmap.GetPixel(i, j);
              bmap.SetPixel(i, j,
              Color.FromArgb(255 - c.R, 255 - c.G, 255 - c.B));
            }
          }
          _currentBitmap = (Bitmap)bmap.Clone();
        }
### 7. Resize
This is resizing the width and height of the image without affecting any pixels of the image so that it does not affect the resolution of the image.

          public void Resize(int newWidth, int newHeight)
          {
            if (newWidth != 0 && newHeight != 0)
            {
              Bitmap temp = (Bitmap)_currentBitmap;
              Bitmap bmap = new Bitmap(newWidth, newHeight, temp.PixelFormat);

              double nWidthFactor = (double)temp.Width / (double)newWidth;
              double nHeightFactor = (double)temp.Height / (double)newHeight;

              double fx, fy, nx, ny;
              int cx, cy, fr_x, fr_y;
              Color color1 = new Color();
              Color color2 = new Color();
              Color color3 = new Color();
              Color color4 = new Color();
              byte nRed, nGreen, nBlue;

              byte bp1, bp2;

              for (int x = 0; x < bmap.Width; ++x)
              {
                for (int y = 0; y < bmap.Height; ++y)
                {

                  fr_x = (int)Math.Floor(x * nWidthFactor);
                  fr_y = (int)Math.Floor(y * nHeightFactor);
                  cx = fr_x + 1;
                  if (cx >= temp.Width) cx = fr_x;
                  cy = fr_y + 1;
                  if (cy >= temp.Height) cy = fr_y;
                  fx = x * nWidthFactor - fr_x;
                  fy = y * nHeightFactor - fr_y;
                  nx = 1.0 - fx;
                  ny = 1.0 - fy;

                  color1 = temp.GetPixel(fr_x, fr_y);
                  color2 = temp.GetPixel(cx, fr_y);
                  color3 = temp.GetPixel(fr_x, cy);
                  color4 = temp.GetPixel(cx, cy);

                  // Blue
                  bp1 = (byte)(nx * color1.B + fx * color2.B);

                  bp2 = (byte)(nx * color3.B + fx * color4.B);

                  nBlue = (byte)(ny * (double)(bp1) + fy * (double)(bp2));

                  // Green
                  bp1 = (byte)(nx * color1.G + fx * color2.G);

                  bp2 = (byte)(nx * color3.G + fx * color4.G);

                  nGreen = (byte)(ny * (double)(bp1) + fy * (double)(bp2));

                  // Red
                  bp1 = (byte)(nx * color1.R + fx * color2.R);

                  bp2 = (byte)(nx * color3.R + fx * color4.R);

                  nRed = (byte)(ny * (double)(bp1) + fy * (double)(bp2));

                  bmap.SetPixel(x, y, System.Drawing.Color.FromArgb
                  (255, nRed, nGreen, nBlue));
                }
              }
              _currentBitmap = (Bitmap)bmap.Clone();
            }
          }
### 8. Rotating and Flipping
Rotating or flipping is also referred to as creating a mirror of a image. This is done in a very simple way by calling the enums available in C#.

        public void RotateFlip(RotateFlipType rotateFlipType)
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          bmap.RotateFlip(rotateFlipType);
          _currentBitmap = (Bitmap)bmap.Clone();
        }
### 9. Crop
To cut out or trim unneeded portions of an image is crop. Here we perform this in 2 steps. First we mention the unneeded part as a semi transparent area. then as the users wish, we crop the image.

        public void DrawOutCropArea(int xPosition, int yPosition, int width, int height)
        {
          _bitmapPrevCropArea = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)_bitmapPrevCropArea.Clone();
          Graphics gr = Graphics.FromImage(bmap);
          Brush cBrush = new Pen(Color.FromArgb(150, Color.White)).Brush;
          Rectangle rect1 = new Rectangle(0, 0, _currentBitmap.Width, yPosition);
          Rectangle rect2 = new Rectangle(0, yPosition, xPosition, height);
          Rectangle rect3 = new Rectangle
          (0, (yPosition + height), _currentBitmap.Width, _currentBitmap.Height);
          Rectangle rect4 = new Rectangle((xPosition + width),
          yPosition, (_currentBitmap.Width - xPosition - width), height);
          gr.FillRectangle(cBrush, rect1);
          gr.FillRectangle(cBrush, rect2);
          gr.FillRectangle(cBrush, rect3);
          gr.FillRectangle(cBrush, rect4);
          _currentBitmap = (Bitmap)bmap.Clone();
        }
Then accordingly, we get the option from the user to crop it or not.

        public void Crop(int xPosition, int yPosition, int width, int height)
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          if (xPosition + width > _currentBitmap.Width)
            width = _currentBitmap.Width - xPosition;
          if (yPosition + height > _currentBitmap.Height)
            height = _currentBitmap.Height - yPosition;
          Rectangle rect = new Rectangle(xPosition, yPosition, width, height);
          _currentBitmap = (Bitmap)bmap.Clone(rect, bmap.PixelFormat);
        }
### 10. Inserting Text, Any Other Images and Shapes
This is just including any required things in the image. This is achieved by the Graphics object of the image.

#### Text

        public void InsertText(string text, int xPosition,
        int yPosition, string fontName, float fontSize,
        string fontStyle, string colorName1, string colorName2)
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          Graphics gr = Graphics.FromImage(bmap);
          if (string.IsNullOrEmpty(fontName))
            fontName = "Times New Roman";
          if (fontSize.Equals(null))
            fontSize = 10.0F;
          Font font = new Font(fontName, fontSize);
          if (!string.IsNullOrEmpty(fontStyle))
          {
            FontStyle fStyle = FontStyle.Regular;
            switch (fontStyle.ToLower())
            {
              case "bold":
                fStyle = FontStyle.Bold;
                break;
              case "italic":
                fStyle = FontStyle.Italic;
                break;
              case "underline":
                fStyle = FontStyle.Underline;
                break;
              case "strikeout":
                fStyle = FontStyle.Strikeout;
                break;
            }
            font = new Font(fontName, fontSize, fStyle);
          }
          if (string.IsNullOrEmpty(colorName1))
            colorName1 = "Black";
          if (string.IsNullOrEmpty(colorName2))
            colorName2 = colorName1;
          Color color1 = Color.FromName(colorName1);
          Color color2 = Color.FromName(colorName2);
          int gW = (int)(text.Length * fontSize);
          gW = gW == 0 ? 10 : gW;
          LinearGradientBrush LGBrush = new LinearGradientBrush(new Rectangle(0, 0, gW, (int)fontSize), color1,
          color2, LinearGradientMode.Vertical);
          gr.DrawString(text, font, LGBrush, xPosition, yPosition);
          _currentBitmap = (Bitmap)bmap.Clone();
        }
        
#### Image

        public void InsertImage(string imagePath, int xPosition, int yPosition)
        {
          Bitmap temp = (Bitmap)_currentBitmap;
          Bitmap bmap = (Bitmap)temp.Clone();
          Graphics gr = Graphics.FromImage(bmap);
          if (!string.IsNullOrEmpty(imagePath))
          {
            Rectangle rect = new Rectangle(xPosition, yPosition,
            i_bitmap.Width, i_bitmap.Height);
            gr.DrawImage(Bitmap.FromFile(imagePath), rect);
          }
          _currentBitmap = (Bitmap)bmap.Clone();
        }
        
#### Shape

          public void InsertShape(string shapeType, int xPosition,
          int yPosition, int width, int height, string colorName)
          {
            Bitmap temp = (Bitmap)_currentBitmap;
            Bitmap bmap = (Bitmap)temp.Clone();
            Graphics gr = Graphics.FromImage(bmap);
            if (string.IsNullOrEmpty(colorName))
              colorName = "Black";
            Pen pen = new Pen(Color.FromName(colorName));
            switch (shapeType.ToLower())
            {
              case "filledellipse":
                gr.FillEllipse(pen.Brush, xPosition,
                yPosition, width, height);
                break;
              case "filledrectangle":
                gr.FillRectangle(pen.Brush, xPosition,
                yPosition, width, height);
                break;
              case "ellipse":
                gr.DrawEllipse(pen, xPosition, yPosition, width, height);
                break;
              case "rectangle":
                default:
                gr.DrawRectangle(pen, xPosition, yPosition, width, height);
                break;
            }
            _currentBitmap = (Bitmap)bmap.Clone();
          }
### Points of Interest
Included Undo Options, Clear Image, Image Information which are quiet simple.
