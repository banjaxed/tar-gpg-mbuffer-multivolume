# tar-gpg-mbuffer-multivolume
A How-To on creating an encrypted tar archive to span multiple DVD volumes

Figuring this out took a while and a few wasted DVDs. Info wasn't easily available on googling.  
  
This example is for an encrypted tar that spans multiple DVDs (4208M volume in this example)   

mbuffer will ask to change volumes and wait for pressing enter  

We will be running commands in two terminals at the same time with a named pipe  

First, we have to make a named pipe:  
``` mknod /tmp/gpg.pipe p ```  

the password in this example is 'foobar', please change to your own.  
Or, you can put the password in pass.txt and change ```--passphrase``` to ```--passphrase-file pass.txt```  
  
in one terminal run:  
``` tar -c -f - Files | gpg --batch -c --passphrase 'foobar' --yes --compress-level 0 -o - | mbuffer -f -i - -o /tmp/gpg.pipe -D 4208M ```  
then, in another terminal run:  
``` cdrecord -data -sao -eject -tsize=4208M -v /tmp/gpg.pipe ```   
cdrecord will burn the first 4208M to disc,  
then terminal 1 mbuffer will ask to enter new volume.  
load second blank disc, hit enter on first terminal and then in second terminal run the cdrecord command again.  
Rinse & repeat until archive is burned.

##for trial runs, use ``` dd if=/tmp/gpg.pipe of=foo.iso1 bs=32768 count=4412407808 ```

#Extract

The mbuffer -n 2 is how many volumes you are about to feed it.  
(dvd blocksize=32768 bytes you may need to know your blocksize)   

```mknod /tmp/gpg.pipe p```  
In first terminal:  
```mbuffer -n 2 -i /tmp/dvd-bak.pipe | gpg --batch --passphrase 'foobar' -d --compress-level 0 -o - | tar xf -```  
In second terminal:  
```dd if=/dev/sr0 of=/tmp/dvd-bak.pipe bs=32768```

I tested this with burning a 8GB archive to 2 DVDs.  
Hope it all works out! Enjoy.

