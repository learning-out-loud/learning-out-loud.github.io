# Key Remapping in Ubuntu

Recently, I got a [Keychron K2 keyboard](https://www.keychron.com/products/keychron-k2-wireless-mechanical-keyboard) with the intention of using it on both Mac and Ubuntu. The keyboard is customizable for Mac and Linux/Windows/Android layout, and in fact comes with a few extra keys to swap the Option and Command keys with Alt and Super keys. As I switch between working with Mac and Linux regularly (using different keyboards), this is not a problem, as long as either the keys are in the same place, i.e., Super and Command, and Alt and Option are the same keys on the keyboard, or I have a Mac keyboard and PC keyboard separately for each. 

However, when using only one keyboard for both, the modifiers for Mac and PC are different. In the lower left corner, for Ubuntu it is  `Control | Super | Alt` and for Mac it is `Control | Option | Command`, as can be seen [here](https://cdn.shopify.com/s/files/1/0059/0630/1017/t/5/assets/pf-79ab923a--k2.jpeg?v=1607047922).

So I had to find a way to stick to one layout and remap the keys when using the other system. In Ubuntu, I could swap Alt and Super mapping with `setxkbmap -option altwin:swap_alt_win`. This means with that command now on Mac I have the same layout I see on the keyboard `Control | Option | Command`, and on Ubuntu I would get `Control | Alt | Super`. Great! 
...But! In Ubuntu, the Window Manager periodically overwrites the key mappings, and I would lose the change!

One way to solve this would be to either find the event that retriggers the overwriting (which I couldn't find!) or periodically rerun the mapping command. While you could just blindly run the command, I wanted to be able to check if the remapping is necessary. `xmodmap -pke` prints the current key mapping, which can be used to see whether the swap has been applied or not. All is left is to put the following Bash script in a file and run it on startup:

```bash
#! /bin/bash

while true
do 
    if [[ ! -z $( xmodmap -pke | grep 134 | grep -i super) ]]
    then 
        logger "Swapping Super and Alt keys." 
        `setxkbmap -option altwin:swap_alt_win`
    fi 
    sleep 5
done
```

`logger` would write to system log in `/var/log/syslog`.

-- 11.05.2022