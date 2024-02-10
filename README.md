import cv2                                                                                    # opencv-conttrib-python hand(motion,tips)
from cvzone.HandTrackingModule import HandDetector                                            # hand tracking
import mouse                                                                                  # tracking of tip of hands
import numpy as np                                                                            # fingers as numbers
import threading                                                                              # to pass arguments
import time                                                                                   # control cursor

detector=HandDetector(detectionCon=0.9,maxHands=1)   # to detection hands function

cap = cv2.VideoCapture(0)
cam_w, cam_h = 640, 480
cap.set(3, cam_w)     # width and height of camera capture
cap.set(4, cam_h)

frameR = 100 # frame size inner rectangle
l_delay = 0
r_delay = 0
double_delay = 0


def l_clk_delay():
    global l_delay
    global l_clk_thread
    time.sleep(1) # setting interval
    l_delay = 0
    l_clk_thread = threading.Thread(target=l_clk_delay)

def r_clk_delay():
    global r_delay
    global r_clk_thread
    time.sleep(1)
    r_delay = 0
    r_clk_thread = threading.Thread(target=r_clk_delay)

def double_clk_delay():
    global double_delay
    global double_clk_thread
    time.sleep(2)
    double_delay = 0
    double_clk_thread = threading.Thread(target=double_clk_delay)

l_clk_thread = threading.Thread(target=l_clk_delay)
r_clk_thread = threading.Thread(target=r_clk_delay)
double_clk_thread = threading.Thread(target=double_clk_delay)

while True:
    success, img=cap.read()    # suc stores boolean is it reading properly or not
    img=cv2.flip(img, 1)        # horizontal frame
    hands, img=detector.findHands(img ,flipType=False)    #extract all information of hands like left, false - already seted the frame horizontal
    cv2.rectangle(img, (frameR, frameR), (cam_w - frameR,cam_h - frameR), (255, 0, 255),2) # frame with rectangle of pink color

    if hands:
        lmlist = hands[0]['lmList']
        ind_x, ind_y = lmlist[8][0], lmlist[8][1]  # index tip
        mid_x, mid_y = lmlist[12][0], lmlist[12][1] # middle tip

        cv2.circle(img, (ind_x, ind_y), 5, (0, 255, 255), 2) # circle with yellow of index
        fingers = detector.fingersUp(hands[0]) # returns array of 5 numbers for fingers
        #print(fingers)

        #mouse movement
        if fingers[1]==1 and fingers[2] == 0 and fingers[0]==1:
            conv_x = int(np.interp(ind_x, (0, cam_w - frameR), (0, 1536)))
            conv_y = int(np.interp(ind_y, (0, cam_h - frameR), (0, 864)))
            mouse.move(conv_x, conv_y) #for screen size

        # mouse button clicks
        if fingers[1] == 1 and fingers[2] == 1 and fingers[0]==1:
            if abs(ind_x-mid_x) < 25 : # distance b/w index & middle
                # left click
                if fingers[4] == 0 and l_delay == 0:
                    mouse.click(button="left")
                    #l_delay = 1
                    #l_clk_thread.start()

                # right click
                if fingers[3] == 1 and r_delay == 0:
                    mouse.click(button="right")
                    #r_delay = 1
                    #r_clk_thread.start()

        #  mouse scrolling
        if fingers[1] == 1 and fingers[2] == 1 and fingers[0] == 0 and fingers[4] == 0:
            if abs(ind_x-mid_x) < 25:
                mouse.wheel(delta=-1)
                #time.sleep(1);
        if fingers[1] == 1 and fingers[2] == 1 and fingers[0] == 0 and fingers[4] == 1:
            if abs(ind_x-mid_x) < 25:
                mouse.wheel(delta=1)
                #time.sleep(1);
        # double mouse click
        if fingers[1] == 1 and fingers[2] == 1 and fingers[3] == 1 and fingers[4] == 1:
            mouse.double_click(button="left")
            #double_delay = 1
            #double_clk_thread.start()

    cv2.imshow("camera Feed", img)                              #img capture from camera
    cv2.waitKey(1)                                              # delay
