# Waveshark 2-Channel RS232 HAT install and setup
This document details the installation and software setup 
process enabling use of the waveshark 2-channel rs232 HAT
on a Raspberry Pi 4+.

## Hardware
	- plug it in
## Software
	### Pi
	    1. open a terminal and run the command
	        - sudo apt update
	        - verify package repositories were updated successfully
		1. edit pi configuration
			- open the firmware config file
				- sudo nano /boot/firmware/config.txt
			- uncomment the following line
				- #dtparam=spi=on
				    - note: the docs did not include this step, but I'm super smart.
			- add the following lines
				# waveshark rs232 HAT configuration
				dtoverlay=sc16is752-spi1,int_pin=24
			- reboot the Pi
				- sudo reboot
		1. after rebooting, verify the SC16IS752 driver has been loaded
		    - run the command ls /dev to verify the presence of the following devices:
		        - gpiochip4
		            - note: the docs reference "gpiochip3", which is incorrect as of 04Sep2025.
		        - ttySC0
		            - serial channel 1
		        - ttySC1
		            - serial channel 2
		1. Support libraries
		    - install wiringpi
		        - ! note: wiringpi was deprecated in 2019 and cannot be installed per the docs
		            - https://www.waveshare.com/wiki/2-CH_RS232_HAT#Install_Libraries
		        - download the package binary and install
		            - note: I used version 3.16. The docs state to use 2.52 because 
		              this is when wiringpi was updated for the Pi 4B. However, the 
		              project was taken over by the community and I proceeded to use 
		              the latest version in testing.
		            - run the commands
	                    cd /tmp
	                    wget https://github.com/WiringPi/WiringPi/releases/download/3.16/wiringpi_3.16_arm64.deb
	                    sudo dpkg -i wiringpi_3.16_arm64.deb
                - run the command 
                    gpio-v 
                  and verify the output contains the correct version number.
		    - install python 3 support
		        - note: waveshark kinda assumes they're the center of your raspberry
		          flavored world here by installing packages system wide. That works
		          for now, but it may not play well with future OS changes (e.g. venv's).
		        - note: install instructions have been updated for python 3. Original:
		            - https://www.waveshare.com/wiki/2-CH_RS232_HAT#Install_Libraries
		        - note: we should be using a python virtual environment to avoid 
		          conflicts with apt. Python 3 doesn't like system wide pip installs. 
		        - run the commands
		            sudo apt install python3-serial
		            python -m pip install RPi.GPIO --break-system-packages
		        - note: I assume the presence of pip3 and python3-serial was already present.
		        
# Test
    - Setup 
        - Test PC (transmitter, )
	        - note: these instructions work on Linux. I'll get you a Windows
	          version, but it'll be something very similar to the below. 
	        - connect a USB to RS232 serial adapter from the PC into channel 1 of the RS232 HAT.
		    - install minicom
			    -	sudo apt update && sudo apt install minicom
		    - find the serial port
			    -	ls /dev/ttyUSB*
			    - most likely to show up as /dev/ttyUSB0 if no other USB devices are present.
		    - note: future sections assume the usb to serial port to be at ttyUSB0
		    - ! Complete Pi setup below before continuing.
		    - install minicom
		        sudo apt install minicom
		    - start a minicom session
		        sudo minicom -D /dev/ttyUSB0 -b 115200
		    - Begin typing and verify what is entered is echoed in the terminal of the Pi.
        - Pi
            - note: the test code provided by waveshark is strictly python 2,
              in a few places. I forked and updated the code, we'll move it later.
            - Fetch test code
                git clone https://github.com/edswangren/ws-rs232-test
            - run the commands
                cd ws-rs232-test/python/examples
                sudo python main.py
            - the script will print the connected serial device to the terminal and wait for input.
