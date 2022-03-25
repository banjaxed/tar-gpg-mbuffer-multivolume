# tar-gpg-mbuffer-multivolume
A How-To on creating an encrypted tar archive to span multiple DVD volumes

Figuring this out took a while and a few wasted DVDs. Info wasn't easily available on googling.  
This example is for an encrypted tar that spans multiple DVDs (4208M volume in this example)   #note: 4208M is not the max a DVD can hold. I was just using it in these tests.

We will be running commands in two terminals at the same time with a named pipe  

First, we have to make a named pipe:  
``` mknod /tmp/gpg.pipe p ```  

the password in this example is 'foobar', please change to your own.  
Or, you can put the password in pass.txt and change ```--passphrase``` to ```--passphrase-file pass.txt```  

#####Create Archive  
in terminal one, run:  
```tar -c -f - Files | gpg --batch -c --passphrase 'foobar' --yes --compress-level 0 -o - | mbuffer -f -i - -o /tmp/gpg.pipe -D 4208M```  
in terminal two, run:  
```cdrecord -data -sao -eject -tsize=4208M -v /tmp/gpg.pipe```   
cdrecord will burn the first 4208M to disc,  
mbuffer will ask to change volumes and wait for pressing enter  
load second blank disc, hit enter on first terminal and then in second terminal run the cdrecord command again.  
Rinse & repeat until archive is burned.

##for trial runs, replace the ```cdrecord``` with ```dd if=/tmp/gpg.pipe of=foo.iso1 bs=32768 count=4412407808```

#####Extract Archive

The ```mbuffer -n 2``` is how many volumes you are about to feed it.  
(dvd blocksize=32768 bytes you may need to know your blocksize)   

```mknod /tmp/dvd-bak.pipe p```  
In first terminal:  
```mbuffer -n 2 -i /tmp/dvd-bak.pipe | gpg --batch --passphrase 'foobar' -d --compress-level 0 -o - | tar xf -```  
In second terminal:  
```dd if=/dev/sr0 of=/tmp/dvd-bak.pipe bs=32768```

dd will error on the last disk since it's likely not 100% full, but don't be alarmed. use ```dd...count=4412407808``` to avoid the error. (4412407808 = 4208M in bytes)

I tested this with burning a 8GB photo catalog split between 2 DVDs.  
Hope it all works out! Enjoy.

