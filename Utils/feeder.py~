#!/usr/bin/python

# Feeds a gcode File to the SphereBot.
# 
# Write a line to the serial device and wait for an "ok" response.

# prerequisite:  http://pyserial.sourceforge.net
#   Installation on Ubuntu: sudo aptitude install python-serial




# Configure:
BAUDRATE = 115200
DEVICE = "COM4"
#DEVICE = "/dev/tty.PL2303-00001004"
#DEVICE = "/dev/tty.PL2303-00004006"
TIMEOUT = 10
STARTUPDELAY = 5

# End configuration



import sys
import serial
import re
import time
from optparse import OptionParser

def y_displacement(x):
    # look into file egg-displace.dat for documentation
    return (0.00795338*x*x + 0.0734545*x + 0.15711)

lastX = 0.0

def correctDisplacement(lineIn):
    # extract x and y
    # calculate new y
    # return line with alter y

    global lastX
    foundY = False

    line = lineIn.upper()
    words = pattern.findall(line)
    for word in words:
        if word[0] == 'X':
            lastX = eval(word[1:])

        if word[0] == 'Y':
            y = eval(word[1:])
            foundY=True

    if foundY:
        y = y + y_displacement(lastX)
    else:
        return lineIn

    lineOut=""
    for word in words:
        if word[0] == 'Y':
            lineOut = lineOut + "Y{0}".format(y)
        else:
            lineOut = lineOut + word

    return lineOut

def penChange(lineIn):
    # Test Line for a Pen change request (M1)
    # If true, wait for user input

    if penChangePattern.match(lineIn):
        raw_input('Change pen ... press <Return> when finished ')
    

######################## Main #########################

parser = OptionParser(usage="usage: %prog [options] gcode-file")
parser.add_option("-e", "--egg-displace", dest="wantDisplaceCorrection",
                  action="store_true", default=False,
                  help="Correct displacement if drawn on a egg. The tip of the egg is pointing right hand.")
parser.add_option("-d", "--dont-send", dest="wantToSend",
                  action="store_false", default=True,
                  help="Dont send GCode to SphereBot")

(options, args) = parser.parse_args()



if len(args) != 1:
    parser.error("incorrect number of arguments: need one gcode file to send to the sphereBot!")


if options.wantDisplaceCorrection:
    pattern = re.compile('([(!;].*|\s+|[a-zA-Z0-9_:](?:[+-])?\d*(?:\.\d*)?|\w\#\d+|\(.*?\)|\#\d+\=(?:[+-])?\d*(?:\.\d*)?)')

penChangePattern = re.compile('^M01')

fileToFeed = args[0]
gcode = open(fileToFeed, "r")

if options.wantToSend:
    print "Creating serial connection, timeout set to ", TIMEOUT, " seconds"
    sphereBot = serial.Serial(DEVICE, BAUDRATE, timeout=TIMEOUT)
    print "Please wait ", STARTUPDELAY, " seconds for Arduino to reset"
    time.sleep(STARTUPDELAY) #Creating a serial connection resets the Arduino, need to wait before sending first line or it will timeout

currentLine = 0.0
lines = gcode.readlines()
totalLines = len(lines)

print "Read ", totalLines, " lines from file"

for line in lines:
    currentLine = currentLine + 1

    if currentLine == 1:
        print "First line might timeout on an Arduino with auto reset, increase the startup delay time. Will continue after timeout in ", TIMEOUT, " seconds"
    
    print "Working on line ", currentLine, " of ", totalLines


    print line, "({0:.1f}%)".format((currentLine / totalLines)*100),

    penChange(line)

    if options.wantDisplaceCorrection:
        line = correctDisplacement(line)
        print ">> ", line,


    if options.wantToSend:
        sphereBot.write(line)
        print "Data sent"
	                
        numOfLines = 0 #Counter to count the number of lines serial.readline() reads.
        
        while True: #Do forever
            #readline() blocks until we see \n or it times out
            response = sphereBot.readline() #Read data from the serial port
            print("Received data: "+ response) #Print out line read
            if (response[:3] == "ok:"):
                #print "Seen ok:, breaking out of loop"
                break
            
            #Incase we don't see an ok: from the Arduino this should breakout after a timeout
            #print "Seen timeout, breaking out of loop"
            break  #Break out of loop

