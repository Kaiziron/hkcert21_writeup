## No Door 1 無門 (100 point, 6 solves, ★☆☆☆☆) [First blood 🩸]
---
Solved by: Kaiziron and grhkm


### Description :
```
Find the hidden room.

https://gather.town/app/sWBohD9YwwnU00Sl/hidden-space

Backup:
https://gather.town/app/H4NHCY3Owcom1iyd/hidden-space
```
---
### Solution (maybe unintended?) :

This challenge is really fun, we have to find the hidden room in gather.town

![](https://i.imgur.com/LcTf1FW.png)

There is a house in the map, but there is no hidden room inside, which is expected because this is a CTF challenge, it won't be that easy

After googling about gather.town, we found a github project called teleporter : https://github.com/michmich112/teleporter
`
A google-chrome extension making gather town teleportation friendly`

However, it does not work properly, maybe because its outdated

![](https://i.imgur.com/W1pf7BI.png)

By looking at the source, it's using the gameSpace object

Then I just check the gameSpace object on the console

![](https://i.imgur.com/TJs9iXP.png)

Copy the whole object to a file, and search for the flag

```
cat gamespace.object.txt  | grep hkcert21{
                        "message": "hkcert21{g4rd3v0ir_c4n_als0_t3l3p0rt}",
                        "message": "hkcert21{g4rd3v0ir_c4n_als0_t3l3p0rt}",
```
Then the flag is found

### Flag :
`hkcert21{g4rd3v0ir_c4n_als0_t3l3p0rt}`

### Intended solution :

However, I think the above solution could be unintended, the intended solution might be actually entering the hidden room

By reading the data inside the gamespace object :

![](https://i.imgur.com/78HJaD1.png)

We see that there is 2 maps, one is Living room which should be the map we entered, and the Secret room should be the hidden room

Then I just change the mapId to to hidden room and teleport to somewhere near the centre
```
gameSpace.mapId = 'Secret'
gameSpace.teleport(20, 20, gameSpace.mapId)
```
![](https://i.imgur.com/aIDY5Zr.png)

Then we finally arrived the hidden room, however the flag isn't visible, I think its due to some bug, just refresh the page, then we can see the flag 

![](https://i.imgur.com/2sLlNLz.png)

---

## No Door 2 幻象 (200 point, 14 solves, ★★☆☆☆) 
---
Solved by: Kaiziron and streamline


### Description :
```
There's must be some part the author can hide the flag.
If you can't solve the challenge, you better take a rest and go for an eye examination.

https://gather.town/app/sWBohD9YwwnU00Sl/hidden-space

Backup:
https://gather.town/app/H4NHCY3Owcom1iyd/hidden-space
```
---
### Solution :

This challenge took us lots of time to do, due to a rabbit hole

![](https://i.imgur.com/8jdS7qC.png)

On the living room map, there is a treasure chest

Opening it, we will find a troll image with the flag format

![](https://i.imgur.com/ICK59dK.png)

We spent lots of time finding flag inside this image, but we found nothing...

We are trolled!

Then I tried to see if there are any hidden image for the treasure chest in the gamespace object, but nothing interesting is found

Then I just extract all image links in the gamespace object and download them all

![](https://i.imgur.com/VzIb59N.png)

Most of the images are small, and I dont think there will be flag, so I focus on larger images such as the map

![](https://i.imgur.com/VfA5fTf.png)

Then I used stegsolve on the map image, and found the flag

![](https://i.imgur.com/FhjOcS6.png)

### Flag :
`hkcert21{hidd3n_f14g_is_in_th3_b3ckgr0und}`

---
