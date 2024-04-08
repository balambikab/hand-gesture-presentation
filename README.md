# hand-gesture-presentation
import os
import cv2
from cvzone.HandTrackingModule import HandDetector
import numpy as np

width, height =1280,720
folderpath = "presentation"
cap = cv2.VideoCapture(0)
cap.set(3,width)
cap.set(4,height)
pathImages=sorted(os.listdir(folderpath),key=len)
#print(pathImages)
#variables

imgNumber= 0
hs,ws=int(120*1),int(213*1)
gestureThreshold=300
buttonPressed=False
buttoncounter=0
buttonDelay=20
annotations=[[]]
annotationnumber=-1
annotationStart=False

#handdetector
detector = HandDetector(detectionCon=0.8,maxHands=1)

while True:
    success, img = cap.read()
    img=cv2.flip(img,1)
    pathFullImage=os.path.join(folderpath,pathImages[imgNumber])
    imgCurrent=cv2.imread(pathFullImage)

    hands, img=detector.findHands(img)
    cv2.line(img,(0,gestureThreshold),(width,gestureThreshold),(0,255,0),10)

    if hands and buttonPressed is False:
        hand=hands[0]
        fingers=detector.fingersUp(hand)
        cx,cy=hand['center']
        lmList=hand['lmList']

        #constrain values for easier drawing
       # indexFinger=lmList[0][0],lmList[0][1]
        xval=int(np.interp(lmList[0][0],[width//2,width],[0,width]))
        yval=int(np.interp(lmList[0][1], [150,height-150], [0, height]))
        indexFinger=xval,yval
        #print(fingers)

        if cy<=gestureThreshold: #if hand is at the height of the face
            annotationStart = False

            #gesture 1-left
            if fingers==[1,0,0,0,0]:
                annotationStart = False
                print("left")

                if imgNumber>0:
                    buttonPressed = True
                    annotations = [[]]
                    annotationnumber = -1

                    imgNumber -=1


            # gesture 2-right
            if fingers == [0, 0, 0, 0, 1]:
                annotationStart = False
                print("right")

                if imgNumber<len(pathImages)-1:
                    buttonPressed = True
                    annotations = [[]]
                    annotationnumber = -1

                    imgNumber += 1



            # gesture 3-pointer
        if fingers == [0, 1, 1, 0, 0]:
            cv2.circle(imgCurrent,indexFinger,12,(0,0,255),cv2.FILLED)
            annotationStart = False
            # gesture 4-drawing
        if fingers == [0, 1, 0, 0, 0]:
            if annotationStart is False:
                annotationStart=True
                annotationnumber+=1
                annotations.append([])
            cv2.circle(imgCurrent, indexFinger, 12, (0, 0, 255), cv2.FILLED)
            annotations[annotationnumber].append(indexFinger)

        else:
            annotationStart=False

        #gesture 5 - erase
        if fingers==[0,1,1,1,0]:
            if annotations:
                #if annotationnumber>=0
                    annotations.pop(-1)
                    annotationnumber-=1
                    buttonPressed=True
    else:
        annotationStart = False
    #button pressed iterations
    if buttonPressed:
        buttoncounter+=1
        if buttoncounter>buttonDelay:
            buttoncounter=0
            buttonPressed=False

    for i in range(len(annotations)):
        for j in range(len(annotations[i])):
            if j!=0:
                cv2.line(imgCurrent,annotations[i][j-1],annotations[i][j],(0,0,200),12)

    #adding web cam img on the slides
    imgSmall=cv2.resize(img,(ws,hs))
    h,w,_=imgCurrent.shape
    imgCurrent[0:hs,w-ws:w]=imgSmall


    cv2.imshow("image",img)
    cv2.imshow("slides",imgCurrent)
    key = cv2.waitKey(1)
    if key == ord('q'):
        break

