may 7 26  
The parts for my 3d printer finally came in and i have started on the process of getting it set up for various projects like cable management, keystone patch panels, blanks, pegboard, etc.

in the google cyber security linux and sql class linux was a good refresher but sql was a bore. i hate that the instructer was just bad. i could tell she wasnt in it.

may 9 26  
network chuck week 1 was alot of the fundementals but i did learn alot more about using packet tracer than i did in any other bootcamt and it made it alot more useful.

after an hour and a half of trying to wrestle with alpine linus i rage quit and went back to linux mint. i know its a skill issue but im also not patient enough to keep fighting it right now

using nano for these. for the uninnitiated nano is kind of like a small terminal inside of the terminal. its technicly a text editer but its easier for me to understand it as terminal inception

may 10 26  
i installed home assistant and matter bridge and mqtt so i can have our alexa act as the vocal portion of the network and the whole thing will be managed by home assistant. now ill be buying new smart appliances (If you want to help they are on the amazon wishlist) since i have changed phones and no longer have the apps for the different appliances i have ill keep the vm asleep till i have the new gear.

may 11 26  
today i started week 2 of the network chuck academy free summer of ccna. and i am a little over halfway through with the google cyber security. the network chuck academy course is alot more entertaining than other networking classes i have taken and i fell like i am grasping alot of it and i will be looking at expanding my network whether professional or it because its very importsnt.

in class they used the military adage as two is one and one is none. they also spoke about the importance of cable labeling and in alot of ways it can be done with various colors

my colorways are (in theory going to be)  
\[enter color code here\]

i have been working on thinking of adding to my projects list and today i think i will work through what i will need in order to set up a landline phone. i also think that when i am done with the google cyber security i will step back and drill down on getting the isc2 cc. i know it isnt the sec+ that everyone is looking for but it is a recognized and respected organization and i fell it will be a good addition to my portfolio which at the moment may or may not be a list of what most people would think is just random once i get the cc i will also start taking the you suck at programming free course so my bash cheatsheet will fill out nicely and i can hopefully stop googling everything while i am in my vm.

one think ive noticed is that network chuck academy breaks each day into 6 tasks a day and perhaps i should continue that over the weekend. its usually 3 small items and 3 small quizzes. perhaps i can do something similar. I'll most likely use this plan to increase the productavity of my lab. perhaps i should start by taking inventory of what has been set up and installed and confugired as well as what needs to be gone.

the study items i use at the moment is linkedin learning, coursera (grow with google), open exam prep, and anki flash cards. i found the flashcards for jeremys it lab which should make this alot easier to both digest and remember. i may replace 2 of the 3 quizzes over the weekend with 10 rounds of flashcards and the last as a 50 questions quiz.

may 12 26

today in the newtork chuck academy class we covered routers. i may take this weekend to go ahead and flash opnsense onto my cisco 5512-x so i can start working with firewall rules more.  
i think a smart way to approach that would be to do a zero trust/ default deny ruleset and adjust the rules as needed instead of trying to map everything out beforehand.

may 14 26

today i took some time away from google again the google class is getting really hard to pay attention to so i will take a few days away. i didnt do any network chuck stuff yesterday because i was at church so today i caught up, luckaly my time in structured cabling and as an electrician made it really easy to get caught up though i have never been able to remember the cabling standards. ill probably need to catch up on them. i also need to start using the anki flashcards more often.

may 16 26

i unstalled the ups and cleaned up the wiring in my server rack using velcro, bread ties, and 3d printed pieces. i now have to troubleshoot why i can not reach my proxmox machine. i also plan to get the new machine setup with either open media vault or truenas scale. i am also contemplating about whether the software handbrake will allow for better cpu transcoding so i can move my jellyfin instance from a dedicated machine to an lxc and free up more space, i am likewise looking into the best firewall rules for opnsense so when i flash the asa5512-x i will already have the legwork done and minimize any vulnerabilities on the network.

while taking the network chuck academy i learned that i am not the only one that needs a visual for understanding especially of concepts that are more complecated. i redid the wireing for the lab and it made proxmox go out. i will have to get it sorted out. after a while the ip address self corrected. i am now working on the primary dns vm. it will run adguard home and nut. as is routine the host os will be linux mint.

may 17 26

innitialized truenas on my second r610 and proceded to create a pool. the pool it small but with todays prices its the best i can do. i have added some hard drives to the wish list should someone want to be a god send.

i set the nas up in raidz2 (raid 6) to gice me a good amount of storage for now.

i promptly mapped my nas to my network and got to work installing nextcloud and immich. nextcloud is a bugger when it comes to the configiguration so im looking into other options

may 18 26

today lets look at modeling protocols. there are two but the one i like to work with is the OSI model. i remember it with the mnumonic please (physical), do (data link), not (network), throw (transport), sausage (session), pizza (presentation), away (application). now tcp/ip shares the physica, data link, network, and transport but in this model the session, and presentation are combined with the application layer making a large application layer. just for knowledge sakes the physical and data link used to be together and called the link layer and network used to just be called internet.

may 19 26

lets do a little more with the osi model. so from the top we have the application layer and this is network aware and when you use an application it will send out a request for the information. then at the presentation layer the data is formated by encapsulation into a "language" the other end can understand. by doing this encapsulation it adds a header. as it travels down to the session layer we need to keep in mind that this is the layer the OS uses to keep its sessions sepperate (it actually has to do with port numbers. if you have been doing any form of home labbing im sure you are familliar with port numbers and how they need to be mapped so that they do not overlap.) this is also where it maintains session data and keep alives. at the transport layer the application will decide what porticol it will use to traverse the internet and the majority of the time the choice is tcp (reliable) and udp (unreliable). once it picks a protocol then it is assigned the port number (there are 65,536 port numbers and i suggest you make a map of them) any port number below 1024 is considered a well known port number (443 is https and 25 is smtp or the simple mail transport protocol). at the network the logical addressing is applied such as the ip address (you may not put the numbers in but your isp has a dns server that it uses or if you are like me you host your own....or two and that will guide you from the words to the site because the dns server has a record that it keeps of all the sites you use and honestly mine is mirrored to opn dns which is hosted by cisco). and as a reminder along the way several headers are added to the data at various layers. at the data link layer you get the mac address added to the data that way it can be added to the routing tables for future use. if you are ever curious and are on a windows pc about what the route looks like to a specific site (for example cisco.com) you can use the command tracert -d cisco.com and it will show you all the ip addresses of the various routers the ping goes through in order to get to its destination. with each hop the various routers rip away the source address and replace it with theit own which is great for security. at the physical layer it will adda a crc (cyclical redindancy check) which is use from point a to point z no information in the data is chanded and with all the headers and footers it is called a frame. your cables are at the physical layer. now along the way your pc did what is called a 3-way handshake where your pc sends a syn request, the server sends a syn ack, and then your pc sends an ack which lets both parties know the drawbridge has been lowered and is ready to be travered across.

for a visual of the data frame it looks kind of like this

\[L2 trailer\]\[DATA\]\[L4 header\]\[L3 header\]\[L2 header\]

and these also have different names at each layer  
L1 = bits  
L2 = frames  
L3 = packets  
L4 = segments  
and then the rest is simply data.

may 20 26

lets go over the three big networking titles. the network architect designs the whole project. he then hands it down to the network engineer is the guy that puts it together and then the network administrator is the one that does the configurations. i bet thats as clear as mud but im trying here. this lesson focuses on the foundation and how you should establish a consistant way of doing things so that in the future you can trouble shoot with ease and keep the network up, after all the foundational habits you develop will be with you in production also. we start with a router and to start that you will need a console connection (in my lab it is a usb adapter hooked to a serial roll over cable). out of the box a switch will work whether it is managed or not but not a router. the router needs to be told how to connect to the network. it will sit there and wait for instructions. when dealing with the terminal you will need a terminal like in the case of mac os or linux (my prefered) or a program called putty (im sure there are others out there that can emulate a terminal but this is what ive used before). the console port will be a bluish color sometimes and be ethernet (rj45) port shaped but it is not for data. it is for configs and thats all it is used for. the cable can be usb or serial with a usb adapter. when you use the terminal you will have lots of options but the speed for most cisco devices is 9600 buid (the class i am taking is cisco specific so alot of the references will be from my cisco notes.) the first command for this lession is show ip interface brief which shows all the info for the ports and vlans and all kinds of cool things.

may 22/26

just a little hint wyou will remember more if you hear something as you read it. us old folks will remember taking turns reading in class and studies have shown that it increases the active recall of information.while i study a tip i use is the google translate function and i have it read to me as i read the work which will help mw remember it. another great and proven method is anki flashcards. if you are anything like me you never learned to study for whatever reason and anki is a great spaced repitition flash card engine that i swear by. on the practical end of things having a standardized way of doing things really helps when it comes to trouble shooting. label both ends and make notes. as you hash it out you will understand better. one example for me is port is always lan from the router (as soon as i stop being lazy and get an sfp module i will be changing that.) but my plan is that in the future i will set up vlan 10 as the poe vlan and it will be tied to port 1 which will uplink to another switch that is poe enabled. you have to play the "what if" game and think of what if i were to do x, y, or z

show running config lets you see what youve done and you can verify what you did

show ip interface brief gives you a clean consise snapshot of the device

ctrl+z lets you quickly exit after you make changes (so will the command end)

exit lets you backout one layer

cisco know not everyone memorizes syntax


to get the device going in privledged mode you input enable (en+tab) will take you to a #

a ? will tell you what commands are available to you 

there are alot of sub modes in cisco so there are any number of ways to do things and having a uniform way of doing things are essential

each mode has different things they can do

to change things globally on the switch you need to be in global configuration mode. to get there you have to be in privledged mode and enter configure terminal. it will be signified by hacing (config)# after the switch name (coincidently you can change the name of the switch by entering hostname and the name you want it to have)

from there you can enter interface config mode (fast ethernet 0/1 for example yould be interface fastethernet 0/1) and it ill have (config-if)# to signify that you are in interface configuration and from there you can put in a description to name the device on that port. and if you want to shut down the switch you will enter shutdown (to bring it back up the command is no shutdown). you can also adjust the speed with the command speed. interface is also where you would assign ip addresses

may 23/26
today starts where yesterday left off. we start with the basic things like a basic cisco config frame work like a naming convention. according to the class the best way to start is to use a basic template and then use it to branch out to what you need. the template i have from my notes is:
1. Hostname
2. Banner MOTD
3. Enable secret
4. Console password and login
5. VTY password and remote access settings
6. Service password-encryption
7. Management IP on VLAN 1
8. Port descriptions
9. Saving the configuration

i plan to add and adapt this for my own uses but here it is so you too can have a basic plan for network setup and config. also be sure to double check any used items are erased as well by using the command write erase. it will be useful when you purchase older gear or decomission your own. 

may 25/26 

I have been working on my new to me nas that i was gifted from a great friend of mine and while adding nextcloud i had a head scratcher but after doing some digging i came across a video from koroma tech which came in clutch for explaining why my instance would not install. its permission issues. go figure. so i ran the below commands and everything was all good

dmin@truenas[~]$ sudo su
[sudo] password for truenas_example: 
root@truenas[/home/truenas_admin]# chown -R www-data:www-data /mnt/example/*

now the web ui is having issues with domain trust. lucally the same video has a solution for that. (i really should reach out to this guy on linkedin and tell him his video was a god send). so i made my way into config and i did some nano work. now it will take me to the sign in page. now if only i can get it to let me log in. let me do some more work on this because it should let me in but no its a persistant permission issue. in the end i have deleted it and will me researching alternatives. i may be replacing nextcloud with a culmonation of various lightweight apps. i may try again in the future but i think for now lets try another path. turns out from what ive seen it is a well known and documented bug that i will be keeping an eye on so i can jump on it when its finally fixed. from what ive seen this bug has been persistant for 2.5 years
boo
