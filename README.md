# Capstone-design
Music block as multi-sensory learning for children with autism

import spidev, time
import  RPi.GPIO as GPIO

spi = spidev.SpiDev()

class Point:

    def __init__(self, xpos, ypos):
        self.xpos = xpos
        self.ypos = ypos
        
    stack = 0
    pre_adc=505
    R=[]
    com_R=[]
    com_R.append(9838.0)
    dp=10117.0
    cnt=255

block_type = []

point=[]
for i in range(9):
    point.append(Point(i%3, int(i/3)))

# def write_pot(input):
#     spi.open(0,1)
#     spi.mode = 0b00
#     spi.max_speed_hz = 1000000
#     lsb = input & 0xFF
#     spi.xfer([0b00010001,lsb])
#     spi.close()

def write_pot(input):
    spi.open(0,1)
    spi.mode = 0b00
    spi.max_speed_hz = 1000000
    lsb = input & 0xFF
    spi.xfer([lsb])
    spi.close()

def analog_read(channel):
    spi.open(0,0)
    spi.mode = 0b00
    spi.max_speed_hz = 1000000
    time.sleep(0.05)
    r = spi.xfer2([1, (0x08+channel)<<4, 0])
    adc_out = ((r[1]&0x03)<<8) + r[2]
    spi.close()
    return adc_out

dic={5:4895,6:5868,7:6861,8:7820,9:8837,10:9855,11:10815,12:11800,13:12765,14:13733}
def judge(r):
    for i in range(5,15):
        if r<=dic[i]+500 and r>dic[i]-500:
            return float(dic[i])
    return r

while(1):
    # a=input()
    # write_pot(a)
    # time.sleep(0.1)
    # print(analog_read(0))
    

    for i in range(9):
        #time.sleep(0.05)
        write_pot(point[i].cnt)
        if i!=0:
            adc=505
        else:
            adc=analog_read(0)
        if abs(adc-point[i].pre_adc)>10:
            time.sleep(0.05)
            adc = analog_read(0)
            if abs(adc-point[i].pre_adc)<10:
                continue
            v = adc*5.1321/1024
            if point[i].pre_adc>adc:
                print("push")
                print("v :",v)
                print("before adc : ",adc)
                point[i].R.append(judge(v/(((5.1321-v)/point[i].dp)-(v/point[i].com_R[-1]))))
                print("Calculated R : ",v/(((5.1321-v)/point[i].dp)-(v/point[i].com_R[-1])))
                point[i].stack+=1
                point[i].com_R.append(1/((1/point[i].com_R[-1])+(1/point[i].R[-1])))
                print("Calculated Com_R : ",point[i].com_R[-1])
                print("Judged R :",point[i].R[-1])
                point[i].cnt=int((point[i].com_R[-1]-60)/(10117.0/256.0))
                #input()
                write_pot(point[i].cnt)
                point[i].dp=60+(10117.0/256.0)*point[i].cnt
                time.sleep(0.05)
                adc=analog_read(0)
                v = adc*5.1321/1024
                print("after v: ",v)
                print("after adc : ",adc)
                print("dp : ",point[i].dp)
                print("cnt : ",point[i].cnt)
                print("stack : ",point[i].stack)
            else:
                print("pop")
                point[i].R.pop()
                point[i].com_R.pop()
                point[i].stack-=1
                if point[i].stack==0:
                    point[i].cnt=255
                    write_pot(255)
                    point[i].dp=10117.0
                    time.sleep(0.05)
                    adc=analog_read(0)
                    print("None")
                else:
                    print("v :",v)
                    print("before adc : ",adc)
                    print("R : ",point[i].R[-1])
                    print("Com_R : ",point[i].com_R[-1])
                    point[i].cnt=int((point[i].com_R[-1]-60)/(10117.0/256.0))
                    write_pot(point[i].cnt)
                    point[i].dp=60+(10117.0/256.0)*point[i].cnt
                    time.sleep(0.05)
                    adc=analog_read(0)
                    v = adc*5.1321/1024
                    print("after v: ",v)
                    print("after adc : ",adc)
                    print("dp : ",point[i].dp)
                    print("cnt : ",point[i].cnt)
                    print("stack : ",point[i].stack)

            #print(point[i].R[-1],point[i].cnt,point[i].dp)   
            print("") 
            point[i].pre_adc=adc
