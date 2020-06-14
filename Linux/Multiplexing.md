
# Multiplexing

### Screen

A screen is essentially another tab. The number that a screen is given when creating it is its PID. You can then search for the process by using htop and typing the PID number to jump to it.

* Create a screen - ```screen -S myscreen```
* List all screens - ```screen -ls```
* Re-attach to a screen - ```screen -r 1234.myscreen```
* Leave a screen (without deleting it) - ```ctrl+a, d``` while connected to the screen press *ctrl+a* at the same time, then let go of *ctrl+a* and press *d*
* Delete/remove a screen - ```ctrl+a, k``` while connected to the screen press *ctrl+a* at the same time, then let go of *ctrl+a* and press *k*

### Tmux

See tmux notes [here](https://rolandw.dev/Notes/Linux/tmux).
