Apollo3 EVB micro_speech Keil project example.

1)
Follow SOME directions at 
"https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/experimental/micro" 
for "Building Portable Reference Code using Make".

Specifically:
i)Open a terminal
ii)Download the TensorFlow source with "git clone https://github.com/tensorflow/tensorflow.git"
iii)Enter the source root directory by running "cd tensorflow"


2)
Unable to successfully run 
"make -f tensorflow/lite/experimental/micro/tools/make/Makefile TARGET=apollo3evb third_party_downloads"
in a Windows ENV with out changes.
Modify "tensorflow/lite/experimental/micro/tools/make/apollo3evb_makefile.inc".

leave only :
  $(eval $(call add_third_party_download,$(AM_SDK_URL),$(AM_SDK_MD5),AmbiqSuite-Rel2.0.0,patch_am_sdk))

comment out all others, EX:
  #$(eval $(call add_third_party_download,$(GCC_EMBEDDED_URL),$(GCC_EMBEDDED_MD5),gcc_embedded,))
  #$(eval $(call add_third_party_download,$(CMSIS_URL),$(CMSIS_MD5),cmsis,))
  #$(eval $(call add_third_party_download,$(AP3_URL),$(AP3_MD5),apollo3_ext,))
  #$(eval $(call add_third_party_download,$(CUST_CMSIS_URL),$(CUST_CMSIS_MD5),CMSIS_ext,))


3)
iv)Download the remaining dependencies by running  
"make -f tensorflow/lite/experimental/micro/tools/make/Makefile TARGET=apollo3evb third_party_downloads".


4)
The directory 
"tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_evb/examples"
should now exist.


5)
Place this directory, "micro_speech", at 
"tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_evb/examples/micro_speech"

5a)
Enter the correct directory for the project by running 
"cd "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_evb/examples"

5b)
Download the Keil project to the exmaples directory with 
"git clone https://github.com/AmbiqMicro/TFLiteMicro_MicroSpeech_Keil_AP3BEVB.git"

5c)
Rename "TFLiteMicro_MicroSpeech_Keil_AP3BEVB" to "micro_speech" with
"mv TFLiteMicro_MicroSpeech_Keil_AP3BEVB micro_speech"


6)
The contents of 
"tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_evb/examples/micro_speech"
should be :
README.txt (This document)
keil (Keil project for micro_speech)
src (Apollo3 EVB specific source files)
Support (supporting files needed to run keil project for AP3B EVB)
SupportMaya (supporting files needed to run keil project for Maya board)


7)
Files must be modified to compile the Keil project.
From "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_evb/examples/micro_speech",
perform the following commands:
cp ./Support/cmsis_armclang.h ../../../../CMSIS/ARM/Include/cmsis_armclang.h;
cp ./Support/am_util_faultisr.c ../../../../utils/am_util_faultisr.c;
sed -i -e '44s/sys\/types/stdint/g' ../../../../../kissfft/kiss_fft.h;


8)
// Apollo3 EVB specific features compile options:
// USE AM_BSP_NUM_LEDS : LED initialization and management per EVB target (# of LEDs defined in EVB BSP) 
// USE_TIME_STAMP : Enable timers and time stamping for debug and performance profiling (customize per application) 
// USE_DEBUG_GPIO : Enable GPIO flag polling for debug and performance profiling (customize per application) 
// USE_MAYA : Enable specific pin configuration and features for AP3B "quarter" sized board

*Keil project points to 
"tensorflow/lite/experimental/micro/examples/micro_speech/main.cc", 
"tensorflow/lite/experimental/micro/examples/micro_speech/apollo3evb/audio_provider.cc", and
"tensorflow/lite/experimental/micro/examples/micro_speech/apollo3evb/command_responder.cc"!!!


9)
Added GPIO flag polling "#if" for debug to 
"tensorflow/tensorflow/lite/experimental/micro/examples/micro_speech/apollo3evb/audio_provider.cc"
"tensorflow/tensorflow/lite/experimental/micro/examples/micro_speech/apollo3evb/command_responder.cc"

Add "USE_DEBUG_GPIO" to compile arguments.


10)
Use logic analyzer (EX : Salea) to observe GPIO flag polling
GPIO assignment to slot 1 & 2 pins:
	am_hal_gpio_pinconfig(31, g_AM_HAL_GPIO_OUTPUT); // Slot1 AN pin
	am_hal_gpio_pinconfig(39, g_AM_HAL_GPIO_OUTPUT); // Slot1 RST pin
	am_hal_gpio_pinconfig(44, g_AM_HAL_GPIO_OUTPUT); // Slot1 CS pin
	am_hal_gpio_pinconfig(48, g_AM_HAL_GPIO_OUTPUT); // Slot1 PWM pin

	am_hal_gpio_pinconfig(32, g_AM_HAL_GPIO_OUTPUT); // Slot2 AN pin
	am_hal_gpio_pinconfig(46, g_AM_HAL_GPIO_OUTPUT); // Slot2 RST pin
	am_hal_gpio_pinconfig(42, g_AM_HAL_GPIO_OUTPUT); // Slot2 CS pin
	am_hal_gpio_pinconfig(47, g_AM_HAL_GPIO_OUTPUT); // Slot2 PWM pin

*Picture of debug w/ EVB is "AP3B_EVB_withGPIO.JPG".


11)
Maya support is included.
TODO : Use makefile to build project

11a)
Copy "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_evb"
to "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_maya" 

11b)
Rename "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_maya/examples/micro_speech"
to "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_maya/examples/micro_speech_maya"

11c)
Update Maya BSP updates.
From "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_maya/examples/micro_speech_maya"
cp SupportMaya/am_bsp.c ../../bsp/.;
cp SupportMaya/am_bsp.h ../../bsp/.;
cp SupportMaya/bsp_pins.src ../../bsp/.; 

11d)
From "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_maya/bsp"
../../../tools/bsp_generator/pinconfig.py bsp_pins.src C >am_bsp_pins.c;
../../../tools/bsp_generator/pinconfig.py bsp_pins.src H >am_bsp_pins.h;

11e)
From "tensorflow/lite/experimental/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/boards/apollo3_maya/bsp/keil"
Recompile "libam_bsp.lib".

11f)
In Keil project at 
"\tensorflow\lite\experimental\micro\tools\make\downloads\AmbiqSuite-Rel2.0.0\boards\apollo3_evb\examples\micro_speech\keil"...
Add "USE_MAYA" to compile arguments.
Remove extra compile options such as "USE_DEBUG_GPIO" and "USE_TIME_STAMP" to compile arguments.


