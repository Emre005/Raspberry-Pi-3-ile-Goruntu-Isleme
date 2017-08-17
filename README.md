<a href="https://mrrol.files.wordpress.com/2016/12/rpi_resim_isleme.jpg"><img class="size-medium wp-image-550 alignleft" src="https://mrrol.files.wordpress.com/2016/12/rpi_resim_isleme.jpg?w=300" alt="rpi_resim_isleme" width="300" height="210" /></a>Daha önce Opencv fonksiyonlarını kullanımıştım ve fonksiyonları Python da kullanmak oldukça kolay. Öncelikle OpenCV kurulması gerekiyor ve bu yazıda Opencv Raspbian kurulumuna değinmeyeceğiz ancak aşağıda verdiğim link sayesinde kurabilirsiniz.Bu yazıda 3 tane uygulama olacak önce sabit resim üzerinde işleme yapacağız sonra aynı işlemleri akan video üzerinde yapıp son olarak kırmızı top takibi ile bitireceğiz.
Kamera olarak RaspberryiPi nin kendi kamera modülünü kullandım.
<!--more-->

<a href="https://mrrol.files.wordpress.com/2016/12/raspberry-cam.jpg"><img class="size-medium wp-image-558 alignleft" src="https://mrrol.files.wordpress.com/2016/12/raspberry-cam.jpg?w=300" alt="raspberry-cam" width="300" height="300" /></a>
Kamera modülünün özellikleri:
<ul>
 	<li>5 megapiksel OV5647 sensör, sabit odaklı lens modülü</li>
 	<li>CCD boyutu: 1/4 inç</li>
 	<li>Kamera çalışma gerilimi 3-5V'dur.</li>
 	<li>Diyafram (F): 2.8</li>
 	<li>Odak uzaklığı: 3.37mm</li>
 	<li>Görüş açısı: 72.4 derece</li>
 	<li>İdeal çözünürlük: 1080p</li>
 	<li>Kamera flat kablosu 60V'u desteklemektedir.</li>
 	<li>2592 × 1944 fotoğraf çözünürlüğü</li>
 	<li>1080p30, 720p60 and 640x480p60/90 video kaydedebilme</li>
 	<li>Boyutlar: 25mm x 24mm x 9mm</li>
</ul>
Kamera modülünü RaspberryPi bağladıp ve çalıştığından emin olmak için aşağıdaki komut ile bir iki fotoğraf alabilirsiniz.

[sourcecode language="c"]
raspistill -o output.jpg
[/sourcecode]

Kamera modülünü python programında kullanabilme için ise "picamera" modülün kurmalıyız. Ancak önce cv görsel ortamında çalışmak için aşağıdaki kodları yazıyoruz.

[sourcecode language="c"]
source ~/.profile
workon cv
[/sourcecode]

"picamera" modülünü yüklemek için;

[sourcecode language="c"]
pip install "picamera[array]"
[/sourcecode]

Kamere için gerekli olan "picamera" modülü kuruldu ancak hemen kamera ile çalışmadan önce hali hazırda var olan bir resim üzerinde Opencv fonksiyonlarını kullanabiliriz.

[sourcecode language="c"]

import cv2
import numpy as np
import os

###################################################################################################
def main():
    imgOriginal = cv2.imread("image.jpg")               # resim aç

    if imgOriginal is None:                             # eğer resim mevcut değilse
        print( "hata : resim bulunamadi \n\n")        # hata
        os.system("pause")                                  # kullanıcı hata mesajını görebilmesi için
        return                                              # main fornksiyonundan çıkış programı bitir.
    # end if

    imgGrayscale = cv2.cvtColor(imgOriginal, cv2.COLOR_BGR2GRAY)        # grayscale dönüştür

    imgBlurred = cv2.GaussianBlur(imgGrayscale, (5, 5), 0)              # blur

    imgCanny = cv2.Canny(imgBlurred, 100, 200)                          # Canny edges

   

    cv2.imshow("imgOriginal", imgOriginal)         # resimleri göster
    cv2.imshow("imgCanny", imgCanny)

    cv2.waitKey()                               #klavyeden bir tuşa balılıncaya kadar bekle

    cv2.destroyAllWindows()                     # bütün pencereleri kapat

    return

###################################################################################################
if __name__ == "__main__":
    main()
[/sourcecode]

https://youtu.be/als_-V4mJvk

Kısa bir ısınma turundan sonra şimdi kamera kullanarak aldığımız frameler aynı Opencv fonksiyonlarını uyguluyoruz.

[sourcecode language="c"]
from picamera.array import PiRGBArray
from picamera import PiCamera
import time
import cv2

def main():
	#kamerayı başlat 
	camera=PiCamera()
	camera.resolution=(640,480)
	camera.framerate=32
	rawCapture=PiRGBArray(camera,size=(640,480))

	#kamera için bekleme
	time.sleep(0.1)

	for frame in camera.capture_continuous(rawCapture,format="bgr",use_video_port=True):
		#resimi temsil eden numPy dizisini alıp imgOriginal içine atıyoruz

		imgOriginal=frame.array
	
		imgGrayscale=cv2.cvtColor(imgOriginal,cv2.COLOR_BGR2GRAY)
	
		imgBlurred = cv2.GaussianBlur(imgGrayscale,(5,5),0)

		imgCanny=cv2.Canny(imgBlurred,100,200)

		cv2.imshow("imgOriginal",imgOriginal)
		cv2.imshow("imgCanny",imgCanny)

		key=cv2.waitKey(1)&0xFF

		rawCapture.truncate(0) # birsonraki frame için rawCapture temizliyoruz.

		if key == ord("q"): # klavyeden q tuşuna basılırsa döngüyü sonlandır.
			return

if __name__ == "__main__":
	    main()
[/sourcecode]

https://www.youtube.com/watch?v=Ti2H3LTlhgU
Son olarak kırmızı top takibi ile bittiriyoruz.

[sourcecode language="c"]
from picamera.array import PiRGBArray
from picamera import PiCamera
import numpy as np
import time
import cv2

def main():
    	#kamerayı başlat 
	camera = PiCamera()
	camera.resolution =(640,480)
	camera.framerate =32
	rawCapture=PiRGBArray(camera,size=(640,480))

	time.sleep(0.1)

	for frame in camera.capture_continuous(rawCapture,format="bgr",use_video_port=True):
                #resimi temsil eden numPy dizisini alıp imgOriginal içine atıyoruz
		imgOriginal = frame.array 
	
		imgHSV=cv2.cvtColor(imgOriginal,cv2.COLOR_BGR2HSV)
	
		imgThreshLow=cv2.inRange(imgHSV,np.array([0,135,135]),np.array([18,255,255]))
		imgThreshHigh=cv2.inRange(imgHSV,np.array([165,135,135]),np.array([179,255,255]))

		imgThresh=cv2.add(imgThreshLow,imgThreshHigh)

		imgThresh=cv2.GaussianBlur(imgThresh,(3,3),2)

		imgThresh = cv2.dilate(imgThresh,np.ones((5,5),np.uint8))
		imgThresh = cv2.erode(imgThresh,np.ones((5,5),np.uint8))

		intRows,intColums = imgThresh.shape

		circles = cv2.HoughCircles(imgThresh,cv2.HOUGH_GRADIENT,5,intRows/4) 
# işlenen resimdeki daireleri circles değişkenine atıyoruz.
#eğer daire bulunuyorsa imgOriginal üzerine çiziliyor
		if circles is not None:
			for circle in circles[0]:
				x,y,radius=circle
				print("top posisyonu x = "+str(x)+", y = "+str(y)+", radius = "+str(radius))
				cv2.circle(imgOriginal,(x,y),3,(0,255,0),-1)
				cv2.circle(imgOriginal,(x,y),radius,(0,0,255),3)
				#end for
			#end if




		cv2.imshow("Frame",imgOriginal)
		cv2.imshow("imgThresh",imgThresh)

		key=cv2.waitKey(1)&0xFF

		rawCapture.truncate(0)

		if key==ord("q"):
			return

if __name__ == "__main__":
        main()

[/sourcecode]

https://www.youtube.com/watch?v=0IvR7j5fRuQ

Opencv Raspbian kurulumu için :
"www.pyimagesearch.com/2015/02/23/install-opencv-and-python-on-your-raspberry-pi-2-and-b/"
