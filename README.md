# openssh-7.5p1-Solaris
Compile openssh-7.5p1 for Solaris X86
This is for the compiling openssh-7.5p1 for Solaris X86 server
All the installations below are to local directories under your home directories.

1.Packages

zlib-1.2.11.tar.gz (From http://zlib.net/ )
openssl-1.0.2l.tar.gz  (From https://www.openssl.org/source/openssl-1.0.2l.tar.gz )
openssh-7.5p1.tar.gz   (From https://mirror.aarnet.edu.au/pub/OpenBSD/OpenSSH/portable/openssh-7.5p1.tar.gz )

2.Working diretories for compiling
Root directory of src and bin files: /home/users/yourhome/openssh/
Openssh directory:  /home/users/yourhome/openssh-7.5p1/
Openssl directory:  /home/users/yourhome/openssl-1.0.2l/
Zlib directory: /home/users/yourhomezlib-1.2.11/

3.Compilers:
Please be sure that you have the C compiler installed. I tested it in Sun's cc
i.e. /usr1/studio12.1/bin/cc

Set enviorment viarables like the following:
PATH=.:/usr1/studio12.1/bin/:$PATH
LD_LIBRARY_PATH=/home/users/yourhome/openssh/openssl-1.0.2l/output/lib:$LD_LIBRARY_PATH

4.Compile zlib :
cd /home/users/yourhome/openssh/zlib-1.2.11
mkdir /output
./configure --prefix=/home/users/yourhome/openssh/zlib-1.2.11/output
make test
make install 

5.Compile Openssl:
5.1. Generate Makefile
cd /home/users/yourhome/openssh/openssl-1.0.2l/
mkdir output
./config shared --prefix=/home/users/yourhome/openssh/openssl-1.0.2l/output --openssldir=/home/users/yourhome/openssh/openssl-1.0.2l/output/openssl

5.2. Modify Makefile 
Changed the out-of-date parameter in Makefile
From -xarchamd64 to -m64

5.3. Make install
make
make test
make install

6.OpenSSH Compiling:
6.1.Generate Makefile
cd /home/users/yourhome/openssh/openssh-7.5p1
mkdir output

Change the configure file as the following:
    #Original
    #if test -n "${need_dash_r}"; then
    #   LDFLAGS="-L/usr/local/ssl/lib -R/usr/local/ssl/lib ${saved_LDFLAGS}"
    #else
    #   LDFLAGS="-L/usr/local/ssl/lib ${saved_LDFLAGS}"
    #fi
    #CPPFLAGS="-I/usr/local/ssl/include ${saved_CPPFLAGS}"

    #Changed to:
    if test -n "${need_dash_r}"; then
        LDFLAGS="-L$with_ssl_dir/lib -R$with_ssl_dir/lib ${saved_LDFLAGS}"
    else
        LDFLAGS="-L$with_ssl_dir/lib ${saved_LDFLAGS}"
    fi
    CPPFLAGS="-I$with_ssl_dir/include ${saved_CPPFLAGS}"


./configure \
  --prefix=/home/users/yourhome/openssh/openssh-7.5p1/output \
  --sysconfdir=/home/users/yourhome/openssh/openssh-7.5p1/output/etc/ssh \
  --with-ssl-dir=/home/users/yourhome/openssh/openssl-1.0.2l/output \
  --with-zlib=/home/users/yourhome/openssh/zlib-1.2.11/output \
    --with-cppflags=-m64     --with-cflags=-m64 --with-ldflags=-m64 \
	--with-privsep-path=/home/users/yourhome/openssh/openssh-7.5p1/output/var/empty	

6.2.Make install
make
make install

7.Configuration file
Create new configuration file: /reuters/pkgs/xxx/.ssh/ssh_my_config

IdentityFile /usr/pkgs/xxx/.ssh/id_rsa_my_private_key
StrictHostKeyChecking no
GssapiDelegateCredentials no
GssapiAuthentication no
GssapiKeyExchange no
PubkeyAuthentication yes
PasswordAuthentication no
Port=[sftp_server_port]
#MACs=hmac-sha2-256
#ProxyCommand /usr/lib/ssh/ssh-socks5-proxy-connect -h [your_proxy_ip] -p 1080 [sftp_server_ip] [sftp_server_port]

8.Verification
You need to do the test in a Solars X86 machine which can connect to target server:

cd /home/users/yourname/openssh/openssh-7.5p1/output/bin
LD_LIBRARY_PATH=/home/users/yourname/openssh/openssl-1.0.2l/output/lib:$LD_LIBRARY_PATH
PATH=/home/users/yourname/openssh/openssh-7.5p1/output/bin:$PATH

The following can show which library the sftp calls:
ldd -d sftp

AIAF Test Server:
./sftp -vvv -o IdentityFile=/usr/pkgs/xxx/.ssh/id_rsa_my_private_key -F /reuters/pkgs/xxx/.ssh/ssh_my_config \
	-S /home/users/yourhome/openssh/openssh-7.5p1/output/bin/ssh  \
    sftp_user_name@ip_address_to_sftp_server
 i.e. sftp_user_name@ip_address_to_sftp_server  like sftp_user@187.98.123.23
	

8.How to generate private/public keys:
ssh-keygen -b 2048 -t rsa -f id_rsa_my_private_key -C my_private_key_comment_20170703
Two files generated: id_rsa_my_private_key(private key), id_rsa_my_private_key.pub(public key)
Send your public key to your target server, and need to add it to server's  /reuters/pkgs/xxx/.ssh/known_hosts
	
