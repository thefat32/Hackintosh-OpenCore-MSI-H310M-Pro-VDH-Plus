# Make a Bootable Clone using Apple's ASR

Based on [Prosoft's article](https://www.prosofteng.com/blog/how-to-create-a-bootable-clone-on-macos?srsltid=AfmBOooP-i-QdZek7l-l2F_MXqPs2B3_W1YlV6n73bMFXb6kzlzD775H)

## Give Permisisons to Terminal

- Enable `Full Disk Access` for `Terminal` in `System Settings => Privacy & Security`

## Prepare target Volume

- Create an empty APFS Volume called `Clone` in the destination APFS Container using Disk Utility

## Copy Volumes using ASR

- Validate the new Volume is mounted in `/Volumes`
    ```sh
    ls /Volumes
    ```

- Initiate ASR copy
    ```sh
    sudo asr restore --no-personalization --source / --target /Volumes/Clone --erase --verbose
    ```

- Validate ASR finished correctly
    ```sh
        Validating target...done
        Validating source...done
        Erase contents of /dev/disk4s1 (/Volumes/Clone)? [ny]: y
        Replicating ....10....20....30....40....50....60....70....80....90....100
        Replicating ....10....20....30....40....50....60....70....80....90....100
        Restored target device is /dev/disk4s1.
        Remounting target volume...done
    Restore completed successfully.
    ```

- The APFS container will contain Volumes copied by ASR with original names
    ```sh
    lucas@Lucass-iMac / % diskutil list
    ...
    /dev/disk4 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +370.6 GB   disk4
                                 Physical Store disk0s2
   1:                APFS Volume Sonoma SSD              10.3 GB    disk4s1
   2:                APFS Volume Sonoma SSD - Data       98.1 GB    disk4s2
   3:                APFS Volume Preboot                 4.0 GB     disk4s3
   4:                APFS Volume Recovery                1.2 GB     disk4s4
    ```

- You can raname Volumes using Disk Utility (Ex. `Sonoma Clone`)

## Copy Cryptexes Directory

If you are running Ventura or Sonoma you will need to copy over the [`Cryptexes` Directory](https://eclecticlight.co/2023/04/05/how-cryptexes-are-changing-macos-ventura/)

- Identify new created `Preboot` partition (Ex. `disk4s3`)

- Mount Preboot
    ```sh
    diskutil mountDisk /dev/disk4s3
    ```
- Copy `Cryptexes` Directory
  ```sh
  sudo cp -av /System/Volumes/Preboot/Cryptexes /Volumes/Preboot
  ```

 ## Update `.disk_label.contentDetails`
 
 ASR Tool creates `.disk_label.contentDetails` in the `Preboot`with the original name of the source disk

- Edit `.disk_label.contentDetails` and use new name (Ex. `Sonoma Clone`)
    ```sh
    sudo vim Volumes/Preboot/<UUID>/System/Library/CoreServices/.disk_label.contentDetails
    ```