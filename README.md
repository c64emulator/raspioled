# raspioled
pythin library for 128 x 64 dot OLED display connected to Raspberry Pi via i2c

<a href="https://github.com/h-nari/raspioled/wiki/images/181217a0.jpg">
<img src="https://github.com/h-nari/raspioled/wiki/images/181217a0.jpg" width="240"></a>

A Python library for a 128 x 64 dot OLED (controller SSD1306) connected to a Raspberry Pi via 1I2C.

I was dissatisfied with the slow drawing speed of Adafruit_SSD1306.

## feature

- Similar to Adafruit_SSD1306, transfer the data of the image drawn by Pillow ((image library) and display it on the OLED.
- High speed because the process of converting Pillow.image.toBytes () to OLED (SSD1306) format is implemented in C language.
- Since the transfer processing of drawing data in i2c is done in the back thread, the Python thread does not wait.
- Also implemented a function to wait for transfer completion. 

## Program example

An example program is shown below.

    #!/usr/bin/python

    from RaspiOled import oled
    from PIL import Image,ImageDraw,ImageFont

    image = Image.new('1',oled.size)  # make 128x64 bitmap image
    draw  = ImageDraw.Draw(image)
    font = ImageFont.truetype(
        font='/usr/share/fonts/truetype/freefont/FreeMono.ttf',
        size=40)
    draw.text((0,0), "Hello", font=font, fill=255)
    oled.begin()
    oled.image(image)


## Installation method
- When used with python2

    $ sudo python setup.py install
    $ sudo pip install Pillow
    
- When used with python3

    $ sudo python3 setup.py install
    $ sudo pip3 install Pillow

## Example of use

Please refer to the sample programs below.

When executing the sample program, if there is a necessary library, install it with pip etc. as appropriate.

## Set the I2C speed to 400kHz

The Raspberry Pi I2C transfer rate is 100kHz by default. As it is, it takes about 100mS to transfer image data, which is not much different from Adafruit_SSD1306.

Increasing the I2C speed to 400kHz will increase the transfer speed by about 4 times. It seems that the speed of I2C is determined by the kernel setting, so change it in /boot/config.txt etc.
Add the following line to /boot/config.txt and restart the Raspberry Pi.

    dtparam=i2c_baudrate=400000
    
## Property

### **size**: 

(128,64)Returns a tuple of screen width and height of oled, ie .

## Method

### **begin(dev="/dev/i2c-1", i2c_addr=0x3c)**

Open the i2c device file and start the drawing subthread.

### **end()**

End the subthread and close the i2c device.

### **clear(update=1,sync=0,timeout=0.5,fill=0,area=(0,0,128,64))**

Clear the drawing buffer.
You can specify the color to fill with fill (0: black, 1: white).
You can specify the area to clear by specifying area. The specification method is (x, y, w, h) or ((x, y), (w, h)).。

If update = 1, it will be transferred to oled immediately after clearing.

By setting sync = 1, wait for the end of transfer. If the default sync = 0, it will return immediately after clearing the buffer without waiting for the transfer to finish.
You can specify the maximum time to wait for the end of the transfer with timeout. The unit is seconds. When a timeout occurs, Python3 raises a TIMEOUT exception and Python2 raises a rapioled.error exception.

### **image(image,dst_area=NULL,src_area=NULL,update=1,sync=0,timeout=0.5)**

Pass the Pillow image object and write it to Oled's drawing buffer.

image is Pillow's Image object. The format must be a bitmap, ie mode must be '1'.

You can specify the area to write on the oled side with dst_area and the area to read from the image side with src_area. If not specified, all areas will be targeted.

The method of specifying the area is one of (x, y), (x, y, w, h), ((x, y), (w, h)). If w and h are not specified, the maximum possible width is assumed.

The update, sync, timeout arguments are the same as the clear method.

### **shift(amount=(-1,0),area=(0,0,128,64),fill=0,update=1,sync=0,timeout=0.5)**

Translates the specified area on the oled by the specified amount.

amount is the amount of movement divided into x and y components. The default is (-1,0), which moves one dot to the left.

area specifies the area to shift. The format is either (x, y, w, h) or ((x, y) (w, h)). If not specified, the entire screen will be moved.

fill specifies the color (0 or 1) that fills the gap created as a result of the move.

update, sync, timeout are the same as the clear method.

### **vsync(timeout=0.5)**

Waits for data transfer to oled after the clear, image_bytes method.
If the data transfer has not been performed, it will be returned immediately.
timeout is the same as the clear command.




    
