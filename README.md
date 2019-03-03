# HowToGetAWSLambdaToRunBinaryFiles
How to get AWS Lambda to run binary files such as ssh

### How to prepare the binary files for execution on AWS Lambda
AWS Lambda runs on the same operating system as the AWS Linux AMI. That operating system is very similar to Centos 7. If you want to run a binary file on AWS Lambda it will need to be compiled to run on the AWS Linux AMI/Centos 7. I recommend doing this on a temporary AWS Linux EC2 instance or a Centos 7 VM/EC2 instance.

### How to package up the binary files for execution on AWS Lambda
AWS Lambda has very few libraries pre installed for you. If your binary file need library files other than the ones already on AWS Lambda you will need to include them in the deployment package. Gather all of the dependencies the binary will need to run.

You can check which files a program needs on a linux box by using the ldd command.
```
$ ldd <path to binary file>

```
If you run the above command on the ssh program you should see something like the following.(This was run on an Ubuntu system)

```
$ ldd /usr/bin/ssh
	linux-vdso.so.1 =>  (0x00007fffc6ce4000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007fd1eb50c000)
	libcrypto.so.1.0.0 => /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 (0x00007fd1eb0c7000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fd1eaec3000)
	libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007fd1eaca9000)
	libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007fd1eaa8e000)
	libgssapi_krb5.so.2 => /usr/lib/x86_64-linux-gnu/libgssapi_krb5.so.2 (0x00007fd1ea844000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd1ea47a000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007fd1ea20a000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fd1eb9de000)
	libkrb5.so.3 => /usr/lib/x86_64-linux-gnu/libkrb5.so.3 (0x00007fd1e9f38000)
	libk5crypto.so.3 => /usr/lib/x86_64-linux-gnu/libk5crypto.so.3 (0x00007fd1e9d09000)
	libcom_err.so.2 => /lib/x86_64-linux-gnu/libcom_err.so.2 (0x00007fd1e9b05000)
	libkrb5support.so.0 => /usr/lib/x86_64-linux-gnu/libkrb5support.so.0 (0x00007fd1e98fa000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fd1e96dd000)
	libkeyutils.so.1 => /lib/x86_64-linux-gnu/libkeyutils.so.1 (0x00007fd1e94d9000)
```

These are all of the library files you will need to include in the deployment package to AWS Lambda.

A short cut to manually copying these files into a directory is to run the following command

```
ldd <path to binary file> | grep "=> /" | awk '{print $3}' | xargs -I '{}' cp -v '{}' /tmp
```
*Source of command is https://www.commandlinefu.com/commands/view/10238/copy-all-shared-libraries-for-a-binary-to-directory

An example of copying the dependencies of ssh to the /tmp directory is below.
```
ldd /usr/bin/ssh | grep "=> /" | awk '{print $3}' | xargs -I '{}' cp -v '{}' /tmp
```
Put the lambda function handler and its dependencies, binary file and all of its dependencies in a directory. Then zip up the files in the directory (not the directory itself).

On a linux machine you can do this with the following command assuming you are currently in the directory

```
zip -r9 ../MyLambdaFunctionDeploymentPackage.zip .
```

### How to execute the binary files in AWS Lambda

There are many ways of executing a binary file in AWS Lambda. An easy way is below. Using Python in this way does not require packaging any dependencies. It is very easy to do.

##### Using Python

Use the subprocess module in Python. You can have it either execute the binary file directly or by calling a shell script first.

An example of calling a shell script is below. (Just modify it if you want to call your binary file directly)
```
import subprocess
def lambda_handler(event, context):
	print(subprocess.check_output(['sh', 'script.sh']))
```
