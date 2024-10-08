# Headless Raspberry Pi OS Flasher
This repository provides a headless method for flashing Raspberry Pi OS onto another bootable device (such as an SD-card, SSD or USB drive) **while the Raspberry Pi is running from another connected device**. The entire process is managed remotely from another computer, such as your PC, via SSH, allowing you to configure and flash the OS without needing direct access to the Raspberry Pi's peripherals. Ideal for users without direct access to the bootable device, this guide automates the flashing process and configures essential settings like Wi-Fi and SSH, ensuring a seamless setup for headless operation. 
This approach is useful when you don't have direct access to the bootable device from your main computer.

### Key Considerations:
- **USB and NVMe Booting Limitations**: If an NVMe SSD is connected, you may not be able to boot directly from USB. Booting from a microSD card first is required to get the OS onto the SSD. Note that this limitation might not exist anymore when you read this.
- **Manual Configurations**: Some post-flashing manual changes may be necessary to ensure proper booting, as the CLI version of Raspberry Pi Imager doesn’t include customized settings.

This repository provides a headless method for flashing Raspberry Pi OS onto a bootable device (such as an SD-card, SSD, or USB drive) **while the Raspberry Pi is running from another connected device**.

## Important Note:

The entire process depends on having a starting point, i.e., a bootable device to initiate the flashing process. If you have access to a computer with a graphical interface, it's highly recommended to initially flash a bootable device using the the GUI version of  [Raspberry Pi Imager](https://www.raspberrypi.com/news/raspberry-pi-imager-imaging-utility). 

### Recommended Process:

1. **Step 1:** Use a computer (e.g., a Windows PC) to flash a USB drive with Raspberry Pi OS using the Raspberry Pi Imager.
   
2. **Step 2:** Boot the Raspberry Pi from this USB drive.

3. **Step 3:** Once the Raspberry Pi is running from the USB drive, use it to flash a microSD card.

4. **Step 4:** After the microSD card is flashed, remove the USB drive, insert an SSD, and use the Raspberry Pi (running from the microSD) to flash the SSD.

This approach allows for a relatively smooth transition between devices and ensures that you can use more permanent storage like an SSD once the initial setup is complete.


## Table of Contents
- [Background](#background)
- [Introduction](#introduction)
- [Key Considerations](#key-considerations)
- [Requirements](#requirements)
- [Customize and Run the Ansible Playbook](#customize-and-run-the-ansible-playbook)
 - [Installing Ansible](#installing-ansible)
 - [Customizing the Playbook](#customizing-the-playbook)
 - [Running the Playbook](#running-the-playbook)
- [Post-Setup Steps](#post-setup-steps)
 - [Connecting to Your New System](#connecting-to-your-new-system)
 - [Fixing Locale Issues](#fixing-locale-issues)
- [Troubleshooting](#troubleshooting)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Background

The traditional method of setting up a Raspberry Pi often involves using a dedicated SD card reader, connecting a monitor, and using a keyboard for initial configuration. However, in many scenarios, these resources may not be readily available, especially in remote or headless setups. This project aims to streamline the process by leveraging existing hardware and network connectivity to perform a complete setup without the need for additional peripherals.

## Introduction

This repository contains an Ansible playbook that automates the process of creating a bootable Raspberry Pi OS device. It's designed for scenarios where you need to set up a Raspberry Pi remotely or without typical setup hardware. The playbook handles several crucial steps:

1. OS flashing: It uses the `rpi-imager` tool to flash the Raspberry Pi OS onto the specified bootable device.
2. Wi-Fi configuration: Sets up the wireless network for immediate connectivity upon boot.
3. SSH setup: Enables and configures SSH for remote access.
4. Initial user creation: Creates a new user account with sudo privileges.
5. System configuration: Sets up locale, timezone, and other system settings.

This approach is ideal for headless installations, IoT projects, or any situation where physical access to the Raspberry Pi is limited.

## Key Considerations

Before using this playbook, there are several important factors to consider:

- **Remote Flashing**: This method is particularly useful when you don't have direct access to the bootable device from your main computer. It allows you to use an intermediary Raspberry Pi or Linux system to perform the setup.

- **Boot Limitations**: There are limitations to booting from USB when an NVMe SSD is connected. In such cases, you may need to flash a microSD card first and boot from it to get the OS onto the SSD. This is due to the boot order preferences in some Raspberry Pi models.

- **Customization Limitations**: The CLI version of Raspberry Pi Imager used in this playbook doesn't include all customized settings that are available in the GUI version. As a result, some manual changes might be necessary after flashing to achieve the exact configuration you need.

- **Network Dependency**: This method relies heavily on network connectivity. Ensure that you have a stable network connection and that you're able to reach the Raspberry Pi after the initial boot.

- **Security Considerations**: The playbook sets up initial SSH access. It's crucial to change passwords and further secure your Raspberry Pi after the initial setup, especially if it will be accessible over the internet.

## Requirements

Before using this playbook, ensure you have the following:

- A Raspberry Pi or Linux system with SSH access to run the playbook
- A bootable device (e.g., NVMe SSD, SD card, or USB drive)
- Pre-downloaded Raspberry Pi OS image (e.g., `raspios_lite_arm64_latest.xz`)
- Network access for Wi-Fi configuration
- Basic understanding of terminal usage and SSH

To download the latest Raspberry Pi OS image, you can use the following commands:

```bash
wget https://downloads.raspberrypi.org/raspios_lite_arm64_latest
mv raspios_lite_arm64_latest raspios_lite_arm64_latest.xz
```

Ensure that you have sufficient permissions to access and write to the bootable device. You may need to run the playbook with sudo privileges.


## Customize and Run the Ansible Playbook

### Installing Ansible

If you haven't installed Ansible yet, follow these steps:

1. Update your system

```bash
sudo apt update && sudo apt upgrade -y
```
2. Install Ansible:

```bash
sudo apt install ansible
```

## Customizing the Playbook:

1. Clone this repository:
```bash
git clone https://github.com/user/headless-rpi-os-flasher.git
cd headless-rpi-os-flasher
```
2. Open the notebook_setup.yml file in your preferred text editor.

3. Modify the following variables in the vars section to match your setup:

```yml
vars:
  bootable_device: /dev/nvme0n1 # Change this to your device path, e.g., /dev/sda, /dev/mmcblk0, etc.
  os_image_path: /home/os_username/raspios_lite_arm64_latest.xz
  wifi_ssid: "YOUR_WIFI_SSID"
  wifi_password: "YOUR_WIFI_PASSWORD"
  new_username: "YOUR_NEW_USERNAME"
  new_password: "YOUR_NEW_PASSWORD"
  country: "US"
  perform_full_wipe: true
  fixed_wait_time: 60 # Adjust waiting time if needed
```
Make sure to replace the placeholders with your actual values.

## Running the Playbook

```markdown
## ⚠️ Warning: Data Loss

Flashing the bootable device will permanently erase all existing data on the specified device. Ensure that you have backed up any important files before proceeding.

Once you run the playbook and confirm the flash operation, all current data on the device (e.g., SD card, SSD, or USB drive) will be overwritten with the Raspberry Pi OS image. This action cannot be undone.

Please double-check the device path (`bootable_device`) in the playbook before starting the flashing process to avoid accidental data loss on the wrong device.
```

1. Ensure you're in the directory containing the playbook.
2. Run the playbook:
```bash
sudo ansible-playbook notebook_setup.yml
```

3. When prompted, type 'yes' to confirm that you want to flash the specified device.

4. Monitor the output for any errors or warnings. The playbook will provide detailed information about each step it's performing.

5. Upon successful completion, you'll see a message indicating that the setup is complete and reminding you to safely remove the bootable device.

## Post-Setup Steps

### Connecting to Your New System

When first connecting to your new system via SSH, you might encounter an SSH key error due to the new OS installation. To resolve this:

1. Remove the old SSH key:
   ```bash
   ssh-keygen -R your_raspberry_pi_ip_address
   ```
   Replace `your_raspberry_pi_ip_address` with your Raspberry Pi's actual IP address.

2. SSH into your Raspberry Pi using the new username and password you set up:
   ```bash
   ssh new_username@your_raspberry_pi_ip_address
   ```

3. After logging in, update your system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### Fixing Locale Issues

If you encounter locale-related issues after booting into your new system, run the following commands:

```bash
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8
sudo reboot
```

You can replace `en_US.UTF-8` with your preferred locale if different.

## Troubleshooting

If you encounter issues while running the playbook or setting up your Raspberry Pi, consider the following troubleshooting steps:

- **Partition Detection Failure**: If the playbook fails to detect the partitions after flashing, try increasing the `fixed_wait_time` variable in the playbook. Some devices may take longer to become ready after flashing.

- **Device Path Issues**: Ensure that the `bootable_device` path is correct for your system. You can use the `lsblk` command to list all block devices and identify the correct path.

- **Image File Not Found**: Check that the Raspberry Pi OS image file exists at the specified `os_image_path`. If you've moved or renamed the file, update the path in the playbook accordingly.

- **Wi-Fi Connection Failure**: If the Raspberry Pi fails to connect to Wi-Fi after booting, double-check the `wifi_ssid` and `wifi_password` variables in the playbook. Ensure they match your network credentials exactly.

- **SSH Connection Issues**: If you can't SSH into the Raspberry Pi after setup, ensure that SSH is enabled in your network and that you're using the correct IP address. You may need to check your router's DHCP leases to find the assigned IP address.

- **Permissions Problems**: If you encounter permission errors while running the playbook, make sure you're using sudo or running as a user with sufficient privileges.

## License

This project is licensed under the MIT License.

## Acknowledgments

- The Raspberry Pi Foundation for their excellent documentation and tools
- The Ansible community for providing a powerful automation framework
- AI-tools for research and formatting (Perplexity, Claude, Chat-GPT).
