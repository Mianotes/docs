# Math formulas

Created: 2026-05-29T19:58:03Z

## Note

To uninstall Arduino App Lab from Debian, you must manually delete the application folder and its hidden configuration files. Since it is distributed as a standalone archive rather than a .deb package, the apt remove command will not work. [[1](https://www.reddit.com/r/arduino/comments/1hs3e2r/how_do_i_completely_uninstall_arduino_id_e_and/)]

🗑️ Step 1: Remove the Application Files [[1](https://www.youtube.com/watch?v=2oHjtoH0QKo)]

By default, the [Official Setup Guide](https://docs.arduino.cc/software/app-lab/setup/linux) recommends installing it in ~/Applications. [[1](https://www.youtube.com/watch?v=2oHjtoH0QKo)]

1. Open your Terminal (Ctrl+Alt+T).

Run the following command to delete the main folder:
bash
rm -rf ~/Applications/arduino-app-lab

1. Use code with caution.
   *If you installed it elsewhere, replace ~/Applications with your specific path.* [[1](https://www.youtube.com/watch?v=GxhDnlud2GU), [2](https://support.arduino.cc/hc/en-us/articles/24333370402076-If-App-Lab-does-not-open-on-Linux), [3](https://www.youtube.com/watch?v=2oHjtoH0QKo)]

🧹 Step 2: Clear Hidden Settings & Data

Arduino applications leave behind large "hidden" folders that store board cores and cache. To perform a complete clean:

Remove the Arduino15 folder (stores board support files):
bash
rm -rf ~/.arduino15

1. Use code with caution.

Remove App Lab specific settings:
bash
rm -rf ~/.arduino-app-lab

1. Use code with caution.

Remove the Shortcut (if created):
If you added it to your application menu, delete the desktop entry:
bash
rm ~/.local/share/applications/arduino-app-lab.desktop

1. Use code with caution.
    [[1](https://ubuntuhandbook.org/index.php/2024/12/install-arduino-ide-ubuntu-debian/), [2](https://www.youtube.com/watch?v=2oHjtoH0QKo)]

🔌 Step 3: Remove the Cloud Agent (Optional) [[1](https://www.youtube.com/watch?v=2oHjtoH0QKo)]

If you also installed the Arduino Cloud Agent to connect your UNO Q to the web browser:

Navigate to the agent folder:
bash
cd ~/ArduinoCloudAgent

1. Use code with caution.

Run the uninstall script:
bash
./uninstall.sh

1. Use code with caution.

Delete the remaining folder:
bash
rm -rf ~/ArduinoCloudAgent

1. Use code with caution.
    [[1](https://askubuntu.com/questions/1108373/how-do-i-remove-arduino-completely)]

⚠️ A Note on Your Projects [[1](https://www.youtube.com/watch?v=2oHjtoH0QKo)]

The commands above will not delete your .py or .ino sketches unless they are inside the arduino-app-lab folder. If you want to keep your code, ensure you have backed up your Arduino folder in your Documents or Home directory before running rm -rf.

If you're also looking to remove the standard Arduino IDE alongside App Lab, this video covers the exact paths for hidden files in Linux:

[Completely UNINSTALL Arduino in Linux Ubuntu -Remove Hidden Files ...](https://www.youtube.com/watch?v=GxhDnlud2GU)

[Aleksandar Haber PhD](https://www.youtube.com/watch?v=GxhDnlud2GU)

[YouTube• 6 Apr 2024](https://www.youtube.com/watch?v=GxhDnlud2GU)

Are you planning to reinstall a different version of the IDE, or are you switching to a different board that doesn't use App Lab? I can help you set up the Arduino CLI if you prefer a lightweight, command-line-only experience on Debian. [[1](https://www.youtube.com/watch?v=2oHjtoH0QKo)]
