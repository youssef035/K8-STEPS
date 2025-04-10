1. Disable Swap

sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab  
![image](https://github.com/user-attachments/assets/71fdd1b2-687d-4265-963f-5e9cc18bb1a6)

