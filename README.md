TGAReader
=========

TGA(Targa) image reader for Java and C.

![alt text](http://3dtech.jp/wiki/index.php?plugin=attach&refer=TGAReader&openfile=TGAReader.png "TGAReader")

## License

Released under the MIT license.

https://github.com/npedotnet/TGAReader/blob/master/LICENSE

## Getting Started

### 1. Add the source code to your project.

**Java**
- Add src/java/TGAReader.java to your project, and modify package statement.

**C**
- Add src/c/tga_reader.{c,h} to your project.

### 2. Create a TGA binary data buffer.

**Java**
```java
	FileInputStream fis = new FileInputStream(new File("test.tga"));
	byte [] buffer = new byte[fis.available()];
	fis.read(buffer);
	fis.close();
```

**C**
```c
#include "tga_reader.h"

FILE *file = fopen("test.tga", "rb");
if(file) {
	int size;
	fseek(file, 0, SEEK_END);
	size = ftell(file);
	fseek(file, 0, SEEK_SET);

	unsigned char *buffer = (unsigned char *)tgaMalloc(size);
	fread(buffer, 1, size, file);
	fclose(file);
}
```

### 3. Create pixels with the RGBA byte order parameter.

ByteOrder|Java|C|Comments
---|---|---|---
ARGB|TGAReader.ARGB|TGA_READER_ARGB|for java.awt.image.BufferedImage, android.graphics.Bitmap
ABGR|TGAReader.ABGR|TGA_READER_ABGR|for OpenGL Texture(GL_RGBA), iOS UIImage

**Java**
```java
	byte [] buffer = ...;
	int [] pixels = TGAReader.read(buffer, TGAReader.ARGB);
	int width = TGAReader.getWidth(buffer);
	int height = TGAReader.getHeight(buffer);
```

**C**
```c
	unsigned char *buffer = ...;
	int *pixels = tgaRead(buffer, TGA_READER_ABGR);
	int width = tgaGetWidth(buffer);
	int height = tgaGetHeight(buffer);
```

### 4. Use created pixels in your application.

#### 4.1. Java OpenGL Application
Sample code to create Java OpenGL texture.

**Java**
```java
	public int createTGATexture() {
	    int texture = 0;
	    
	    try {
	        FileInputStream fis = new FileInputStream(path);
	        byte [] buffer = new byte[fis.available()];
	        fis.read(buffer);
	        fis.close();

	        int [] pixels = TGAReader.read(buffer, TGAReader.ABGR);
	        int width = TGAReader.getWidth(buffer);
	        int height = TGAReader.getHeight(buffer);

	        int [] textures = new int[1];
	        gl.glGenTextures(1, textures, 0);

	        gl.glEnable(GL_TEXTURE_2D);
	        gl.glBindTexture(GL_TEXTURE_2D, textures[0]);
	        gl.glPixelStorei(GL_UNPACK_ALIGNMENT, 4);

	        IntBuffer texBuffer = IntBuffer.wrap(pixels);
	        gl.glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, texBuffer);

	        gl.glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	        gl.glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
	        gl.glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	        gl.glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	        
	        texture = textures[0];
	    }
	    catch(Exception e) {
	        e.printStackTrace();
	    }
	    
	    return texture;
	}
```

#### 4.2. Java Application
Sample code to create java.awt.image.BufferedImage.

**Java**
```java
    private static JLabel createTGALabel(String path) throws IOException {

        FileInputStream fis = new FileInputStream(path);
        byte [] buffer = new byte[fis.available()];
        fis.read(buffer);
        fis.close();

        int [] pixels = TGAReader.read(buffer, TGAReader.ARGB);
        int width = TGAReader.getWidth(buffer);
        int height = TGAReader.getHeight(buffer);
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
        image.setRGB(0, 0, width, height, pixels, 0, width);

        ImageIcon icon = new ImageIcon(image.getScaledInstance(128, 128, BufferedImage.SCALE_SMOOTH));
        return new JLabel(icon);
    }
```

For more details, please refer to the sample project.

https://github.com/npedotnet/TGAReader/tree/master/samples/TGASwingBufferedImage

#### 4.3. Android OpenGL Application
Sample code to create Android OpenGL texture.

**Java**
```java
	import static javax.microedition.khronos.opengles.GL10;

	public int createTGATexture(GL10 gl, String path) {
	    int texture = 0;
	    try {
	        
	        InputStream is = getContext().getAssets().open(path);
	        byte [] buffer = new byte[is.available()];
	        is.read(buffer);
	        is.close();
	        
	        int [] pixels = TGAReader.read(buffer, TGAReader.ABGR);
	        int width = TGAReader.getWidth(buffer);
	        int height = TGAReader.getHeight(buffer);
	        
	        int [] textures = new int[1];
	        gl.glGenTextures(1, textures, 0);
	        
	        gl.glEnable(GL_TEXTURE_2D);
	        gl.glBindTexture(GL_TEXTURE_2D, textures[0]);
	        gl.glPixelStorei(GL_UNPACK_ALIGNMENT, 4);

	        IntBuffer texBuffer = IntBuffer.wrap(pixels);
	        gl.glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, texBuffer);
	        
	        gl.glTexParameterx(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	        gl.glTexParameterx(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
	        gl.glTexParameterx(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	        gl.glTexParameterx(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	        
	        texture = textures[0];
	        
	    }
	    catch (IOException e) {
	        e.printStackTrace();
	    }
	    
	    return texture;
	}
```

For more details, please refer to the sample project.

https://github.com/npedotnet/TGAReader/tree/master/samples/TGAGLViewer_Android

#### 4.4. Android Application
Sample code to create android.graphics.Bitmap.

**Java**
```Java
    private Bitmap createTGABitmap(String path) {
        Bitmap bitmap = null;
        try {
            InputStream is = getAssets().open(path);
            byte [] buffer = new byte[is.available()];
            is.read(buffer);
            is.close();
            
            int [] pixels = TGAReader.read(buffer, TGAReader.ARGB);
            int width = TGAReader.getWidth(buffer);
            int height = TGAReader.getHeight(buffer);
            
            bitmap = Bitmap.createBitmap(pixels, 0, width, width, height, Config.ARGB_8888);
        }
        catch(Exception e) {
            e.printStackTrace();
        }
        return bitmap;
    }
```
For more details, please refer to the sample project.

https://github.com/npedotnet/TGAReader/tree/master/samples/TGABitmapViewer_Android

#### 4.5. iOS OpenGL Application
Sample code to create iOS OpenGL texture.

**Objective-C**
```objc
	- (GLuint)createTGATexture:(NSString *)path {
	    
	    GLuint texture = 0;
	    
	    FILE *file = fopen([path UTF8String], "rb");
	    if(file) {
	        fseek(file, 0, SEEK_END);
	        int size = ftell(file);
	        fseek(file, 0, SEEK_SET);
	        
	        unsigned char *buffer = (unsigned char *)tgaMalloc(size);
	        fread(buffer, 1, size, file);
	        fclose(file);
	        
	        int width = tgaGetWidth(buffer);
	        int height = tgaGetHeight(buffer);
	        int *pixels = tgaRead(buffer, TGA_READER_ABGR);
	        
	        tgaFree(buffer);
	        
	        glGenTextures(1, &texture);
	        glEnable(GL_TEXTURE_2D);
	        glBindTexture(GL_TEXTURE_2D, texture);
	        glPixelStorei(GL_UNPACK_ALIGNMENT, 4);
	        glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, pixels);
	        
	        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
	        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	        
	        tgaFree(pixels);
	    }
	    
	    return texture;

	}
```

For more details, please refer to the sample project.

https://github.com/npedotnet/TGAReader/tree/master/samples/TGAGLViewer_iOS

#### 4.6. iOS Application
Sample code to create iOS UIImage.

**Objective-C**
```objc
	- (UIImage *)createTGAImage:(NSString *)path {
	    
	    FILE *file = fopen([path UTF8String], "rb");
	    if(file) {
	        fseek(file, 0, SEEK_END);
	        int size = ftell(file);
	        fseek(file, 0, SEEK_SET);
	        
	        unsigned char *buffer = (unsigned char *)tgaMalloc(size);
	        fread(buffer, 1, size, file);
	        fclose(file);
	        
	        int width = tgaGetWidth(buffer);
	        int height = tgaGetHeight(buffer);
	        int *pixels = tgaRead(buffer, TGA_READER_ABGR);
	        
	        tgaFree(buffer);
	        
	        CGColorSpaceRef colorSpaceRef = CGColorSpaceCreateDeviceRGB();
	        CGBitmapInfo bitmapInfo = (CGBitmapInfo)kCGImageAlphaLast;
	        CGDataProviderRef providerRef = CGDataProviderCreateWithData(NULL, pixels, 4*width*height, releaseDataCallback);
	        
	        CGImageRef imageRef = CGImageCreate(width, height, 8, 32, 4*width, colorSpaceRef, bitmapInfo, providerRef, NULL, 0, kCGRenderingIntentDefault);
	        
	        UIImage *image = [[UIImage alloc] initWithCGImage:imageRef];
	        
	        CGColorSpaceRelease(colorSpaceRef);
	        
	        return image;
	    }
	    
	    return nil;
	    
	}

	static void releaseDataCallback(void *info, const void *data, size_t size) {
		tgaFree((void *)data);
	}
```
For more details, please refer to the sample project.

https://github.com/npedotnet/TGAReader/tree/master/samples/TGAImageViewer_iOS

### 5. Free allocated memory (C language Only)

**C**
```c
	unsigned char *buffer = ...;
	int *pixels = tgaRead(buffer, TGA_READER_ABGR);
	if(pixels) {
		tgaFree(pixels);
	}
	tgaFree(buffer);
```

## Memory Management (C language Only)

If you have your memory management system, please customize tgaMalloc() and tgaFree().

**C**
```c
	void *tgaMalloc(size_t size) {
		return malloc(size);
	}

	void tgaFree(void *memory) {
		free(memory);
	}
```

## Supported
- Colormap(Indexed) Image, RGB Color Image, Grayscale Image
- Run Length Encoding
- Colormap origin offset
- Image origin(LowerLeft, LowerRight, UpperLeft, UpperRight)

## Unsupported
- Image Type 0, 32, 33
- 16bit RGB Color image
- X/Y origin offset of image


Thank you for reading through. Enjoy your programming life!
