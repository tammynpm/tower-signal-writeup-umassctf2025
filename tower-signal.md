The first step in any steganography challenge should be checking the metadata. Metadata might give away vital clues, like the tool used to encode the data or the baud rate for decoding serial audio.  

```
debian@myLAPTOP:/tmp$ exiftool last_output.wav
ExifTool Version Number     	: 12.57
File Name                   	: last_output.wav
Directory                   	: .
File Size                   	: 38 MB
File Modification Date/Time 	: 2025:04:20 20:32:29-04:00
File Access Date/Time       	: 2025:04:20 20:32:27-04:00
File Inode Change Date/Time 	: 2025:04:20 20:32:29-04:00
File Permissions            	: -rwxr-xr-x
File Type                   	: WAV
File Type Extension         	: wav
MIME Type                   	: audio/x-wav
Encoding                    	: Microsoft PCM
Num Channels                	: 2
Sample Rate                 	: 48000
Avg Bytes Per Sec           	: 192000
Bits Per Sample             	: 16
Comment                     	: Try 100 > 50 > 30
Software                    	: Lavf59.27.100
Duration                    	: 0:03:20

```

```
debian@myLAPTOP:/tmp$ minimodem --rx 100 -f half_aa.wav
### CARRIER 100 @ 1250.0 Hz ###
VEFHXzA1ID0gWlhKZmFBbz0KVEFHXzAyID0gWVhOb1h3bz0KVEFHXzAwID0gVlUxQlV3bz0KVEFH

### NOCARRIER ndata=77 confidence=9.567 ampl=0.936 bps=100.00 (rate perfect) ###
debian@myLAPTOP:/tmp$
```

Using this one command \`minimodem\` you can read and write an audio file at a specific baud rate. In serial communication, audio is transmitted bit-by-bit at a fixed rate (baud rate). If you decode with a mismatched baud rate, you'll get garbled output like:

```
debian@myLAPTOP:/tmp$ minimodem --rx 120 -f half_aa.wav
### CARRIER 120 @ 1250.0 Hz ###
,

�\Н��4�j���E��ϱ�[����;�
��j��t�j���
### NOCARRIER ndata=81 confidence=2.623 ampl=0.854 bps=119.27 (0.6% slow) ###
```

Note that, \`100, 50, or 30\` are not the common baud rates. I used these numbers just because they are easy to implement.   
Back to the second and third parts of the flag, 

```
debian@myLAPTOP:/tmp$ minimodem --rx 50 -f half_ab.wav
### CARRIER 50.00 @ 1590.0 Hz ###
XzA0ID0gWDNSdmR3bz0KVEFHXzAxID0gVTN0bWJBbz0KVEFHXzA2ID0gWlhKMGVnbz0KVEFHXzAz

### NOCARRIER ndata=77 confidence=11.560 ampl=0.984 bps=50.00 (rate perfect) ###

```

```
debian@myLAPTOP:/tmp$ minimodem --rx 30 -f half_ac.wav
### CARRIER 30.00 @ 1590.0 Hz ###
ID0gWVhOdGNnbz0K
```

```
debian@myLAPTOP:/tmp$ echo VEFHXzA1ID0gWlhKZmFBbz0KVEFHXzAyID0gWVhOb1h3bz0KVEFHXzAwID0gVlUxQlV3bz0KVEFH | base64 --decode
TAG_05 = ZXJfaAo=
TAG_02 = YXNoXwo=
TAG_00 = VU1BUwo=
```

```
debian@myLAPTOP:/tmp$ echo XzA0ID0gWDNSdmR3bz0KVEFHXzAxID0gVTN0bWJBbz0KVEFHXzA2ID0gWlhKMGVnbz0KVEFHXzA
z | base64 --decode
TAG_04 = X3Rvdwo=
TAG_01 = U3tmbAo=
TAG_06 = ZXJ0ego=
TAG_03
```

```
 = YXNtcgo=
```

Put them in together: 

```
TAG_05 = ZXJfaAo=
TAG_02 = YXNoXwo=
TAG_00 = VU1BUwo=
TAG_04 = X3Rvdwo=
TAG_01 = U3tmbAo=
TAG_06 = ZXJ0ego=
TAG_03 = YXNtcgo=
```

Rearrange the tags:

```
TAG_00 = VU1BUwo=
TAG_01 = U3tmbAo=
TAG_02 = YXNoXwo=
TAG_03 = YXNtcgo=
TAG_04 = X3Rvdwo=
TAG_05 = ZXJfaAo=
TAG_06 = ZXJ0ego=

```

Decode base64 once again, we get: 

```
UMAS
S{fl
ash_
asmr
_tow
er_h
ertz}
```

The complete flag is \`UMASS{flash\_asmr\_tower\_hertz}\`

