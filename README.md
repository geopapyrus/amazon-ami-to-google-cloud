1) Add the following line to /etc/hosts

```169.254.169.254    metadata.google.internal metadata```


2) Add the following to  /etc/rc.local

```
curl --silent --max-time 10 --connect-timeout 3 -H "Metadata-flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/hostname | awk -F. '{print $1}' > /etc/hostname
```

3) mount a new disk as big as the server at /mnt/extra

4) Create the file  /mnt/extra/menu.lst with

```
default     0

timeout     0

hiddenmenu

title       Amazon Linux 2016.09 (4.4.23-31.54.amzn1.x86_64)

root        (hd0)

kernel      /boot/vmlinuz-4.4.23-31.54.amzn1.x86_64 root=/dev/sda1 console=tty1 console=ttyS0

initrd      /boot/initramfs-4.4.23-31.54.amzn1.x86_64.img
```

5) Create the file /mnt/extra/fstab with

```
/dev/sda1 / ext4 defaults 1 1

none /dev/pts devpts defaults 0 0
```


6) Generate the X.509 certificate for your Amazon account and store the certificate and keys on /mnt/extra/cert.pem and /mnt/extra/pk.pem respectively.

7)  Bundle up the Amazon EBS volume.

```
ec2-bundle-vol -c /mnt/extra/cert.pem -k /mnt/extra/pk.pem -u 6584-1847-4319 -r x86_64 --no-filter --exclude /mnt/extra --grub-config /mnt/extra/menu.lst --fstab /mnt/extra/fstab -d /mnt/extra/
```

8) Create a sparse tarball

```
mv image disk.raw

tar -Sczf aws-image.tar.gz disk.raw
```

9) Upload to a bucket

```
~/google-cloud-sdk/bin/gsutil cp aws-image.tar.gz gs://[BUCKET_NAME]

( create the bucket if it doesn't exist: ~/google-cloud-sdk/bin/gsutil mb gs://[BUCKET_NAME])
```

10) Import the image into Compute Engine:

```
~/google-cloud-sdk/bin/gcloud compute images create amazon-linux-ami \
    --source-uri gs://[BUCKET_NAME]/aws-image.tar.gz
```
