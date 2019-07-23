# Hash Cracking with Free GPU
Hashcat installation on Google colab with tesla k80
 >Google Colaboratory is a free Jupyter notebook environment that requires no setup and runs entirely in the cloud.

>With Colaboratory you can write and execute code, save and share your analyses, and access powerful computing resources, all for free from your browser.
[Welcome to Colab](https://colab.research.google.com/notebooks/welcome.ipynb)

---------- 
###  Mount to Google Drive


```python
from google.colab import drive
drive.mount('/content/gdrive')
```

### Connection to Google Colab with SSH on NGROK
```python
#Generate root password
import random, string
password = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(20))

#Download ngrok
! wget -q -c -nc https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
! unzip -qq -n ngrok-stable-linux-amd64.zip
#Setup sshd
! apt-get install -qq -o=Dpkg::Use-Pty=0 openssh-server pwgen > /dev/null
#Set root password
! echo root:$password | chpasswd
! mkdir -p /var/run/sshd
! echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
! echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
! echo "LD_LIBRARY_PATH=/usr/lib64-nvidia" >> /root/.bashrc
! echo "export LD_LIBRARY_PATH" >> /root/.bashrc

#Run sshd
get_ipython().system_raw('/usr/sbin/sshd -D &')

#Ask token
print("Copy authtoken from https://dashboard.ngrok.com/auth")
import getpass
authtoken = getpass.getpass()

#Create tunnel
get_ipython().system_raw('./ngrok authtoken $authtoken && ./ngrok tcp 22 &')
#Print root password
print("Root password: {}".format(password))
#Get public address
! curl -s http://localhost:4040/api/tunnels | python3 -c \
    "import sys, json; print(json.load(sys.stdin)['tunnels'][0]['public_url'])"
```

### SSH Keygen 
```python
!ssh-keygen -t rsa
%cd /root/.ssh/
!cat id_rsa.pub >> authorized_keys && chmod 600 authorized_keys && cat id_rsa 
```


### Hashcat Installation	
```bash
ls /usr/bin/ | grep -i hash
```
Commandi ile kontrol edip hashcate ait dosyalar kaldıysa silicez.

```bash
wget https://hashcat.net/files/hashcat-5.1.0.7z
sudo apt install p7zip
sudo p7zip -d hashcat-5.1.0.7z
cd hashcat-5.1.0
ls /usr/bin/ | grep -i hash			##check
sudo cp hashcat64.bin /usr/bin/
sudo ln -s /usr/bin/hashcat64.bin /usr/bin/hashcat	
sudo cp -Rv OpenCL/ /usr/bin/
sudo cp hashcat.hcstat2 /usr/bin/
sudo cp hashcat.hctune /usr/bin/
```



### Hashcat Benchmark

![Benchmark](https://github.com/semihucann/hash_cracking_with_gpu/blob/master/benchmark.PNG?raw=true)
