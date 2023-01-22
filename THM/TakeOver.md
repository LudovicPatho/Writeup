# TakeOver - TryHackMe
This challenge revolves around subdomain enumeration.

Link : https://tryhackme.com/room/takeover

## Step 1
First of all, you have to add the vhost in the /etc/hosts file of your computer

````bash
echo "10.10.86.212 futurevera.thm" >> /etc/hosts
````

## Step 2 
Let's start with the usual nmap : 
````bash
nmap 10.10.86.212 -A
````
![9daadd6045bf179429e7d7c796c5cea0.png](:/672bae36bc5044d4947251d7d960f56e)
We can see that ports 22, 80 and 443 are open.

## Step 3
On the address https://futurevera.thm, you can see a site but I did not find anything wrong. The same http address redirects to the https site.

But the room description gives a clue : 
> *This challenge revolves around subdomain enumeration.*

So we will be able to enumerate the subdomains with ffuf (Note that you can also do it with gobuster)

````bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt -H "Host: FUZZ.futurevera.thm" -u http://10.10.86.212
````
![76ec94f2f1dfada4fff8c3192d24cd50.png](:/a9dba3dbf564467b8c6ca736e2ec69fd)

You can filter the result with the -fs flag to exclude results with a size of 0

````bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt -H "Host: FUZZ.futurevera.thm" -u http://10.10.86.212 -fs 0
````
![3055bb2a019551f116127427043921d7.png](:/c7a2f33c95834782b630bd01804e5569)

So we discovered 2 subdomains, but when we go to them, we can see this message.

````bash
curl --header "Host: portal.futurevera.thm" 10.10.86.212 -k
````

![d6e920f33a209ed20edde8fea6b68e68.png](:/ab8ed5f4390b4856acc842d90a2c5451)

Same with payroll subdomain
````bash
curl --header "Host: payroll.futurevera.thm" 10.10.86.212 -k
````
![b6b6b092dee58a0253fe323fceccae70.png](:/44dd1f30e9da46259de238aa633030be)

Nothing usable for the moment.

Our previous Fuzzing was in http. Let's do the same with https.
````bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt -H "Host: FUZZ.futurevera.thm" -u https://10.10.86.212 
````

![ff2b4998f8541326a3bff6d00945b166.png](:/b75fe732791d443aafa989e848ad46ae)

Again, we can filter our results by excluding files with a size of 4605 
````bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt -H "Host: FUZZ.futurevera.thm" -u https://10.10.86.212 -fs 4605
````

![f730c25e4c4deb43454cc5786961ea85.png](:/50ffba57951b4909bc5a179fe85c7c18)

These subdomains seem to have a few things, let's go see what they are. To do this, we need to add these subdomains to the /etc/hosts file. 

````bash
echo "10.10.86.212 blog.futurevera.thm support.futurevera.thm" >> /etc/hosts
````

**Blog :** 
![afbf958e19554683a38316459fdb6303.png](:/c39fd624a4d74f47a5af8a011e061087)
Nothing interesting.

**Support :**
![6fad6cd134f2732a03fec56534c68bb5.png](:/fae89b50252e4a4b8000bea458d6f95a)

It says (like in the task description) that they are rebuilding the support website.

And checking the ssl certificate  
*(Click on the cadena -> Connection not secure -> Remove Exception)*
![205993328e62e361e09a107020a97a58.png](:/f339066d36024e45b7b953403db47060)

We can see that there is a "Subject Alt Name".

![e19440ae140837fda3a9d9b21e137e89.png](:/b2143c8a346941c088842d19a3027b7f)
> secrethelpdesk934752.support.futurevera.thm

Let's also add it to the hosts file

````bash
echo "10.10.86.212 secrethelpdesk934752.support.futurevera.thm" >> /etc/hosts
````

Open the browser, and go to the address.

If we go in https, we fall on the main site, but if we connect on it in http, we are redirected to the flag :) 

![7f68443211bf487f753bdbaef4f03b77.png](:/e022370df6774c0a86238b819e8228df)

## Conclusion:
This is a box that is considered easy, but nevertheless I lost a lot of time because I had forgotten to check the ssl certificate which is a basic check to do under normal circumstances. So I would say **Never forget to do the basic checks**.
