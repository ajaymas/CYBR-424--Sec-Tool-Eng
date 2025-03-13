# **Practical Experimentation: SSH Public Key Authentication on Kali Linux**

## **Objective**  
Students will learn how to:  
✅ Generate an SSH key pair  
✅ Configure SSH authentication using public keys  
✅ Log in to a remote machine using SSH keys  
✅ Test their understanding with CTF-style questions  

---

## **🛠 Step 1: Generate an SSH Key Pair (Client Side - Kali Linux)**
Run the following command to generate a key pair:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

### **Explanation:**  
- `-t rsa`: Generates an **RSA** key pair.  
- `-b 4096`: Creates a **4096-bit** key (stronger security).  
- `-f ~/.ssh/id_rsa`: Saves the private key in the `~/.ssh/` directory.  
- `-N ""`: Creates the key without a passphrase for easier testing (not recommended for production).  

```
remnux@remnux:~/Desktop/SSH$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
Generating public/private rsa key pair.
Created directory '/home/remnux/.ssh'.
Your identification has been saved in /home/remnux/.ssh/id_rsa
Your public key has been saved in /home/remnux/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:AT3vyFpaUMvl9r17C7gZEev57hEaVzmkzHGXszKZEl8 remnux@remnux
The key's randomart image is:
+---[RSA 4096]----+
|      ..     . oo|
|       .+ ..o =Eo|
|       o.* .o+++o|
|      . o.+.o*...|
|       oS+ =.+o  |
|        = o O o  |
|       =   * o . |
|      o     = + .|
|           oo+.+.|
+----[SHA256]-----+

```




#### **Verify Key Pair Creation:**
```bash
ls -l ~/.ssh/
```
🔹 You should see two files:  
- `id_rsa` (Private Key)  
- `id_rsa.pub` (Public Key)  

---

## **🛠 Step 2: Copy the Public Key to the Server**
Use `ssh-copy-id` to transfer the public key to the target server:

```bash
ssh-copy-id user@target-ip
```

If `ssh-copy-id` is not available, manually copy the key:

```bash
cat ~/.ssh/id_rsa.pub | ssh user@target-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### **Explanation:**  
- This command appends the public key (`id_rsa.pub`) to the `authorized_keys` file on the server.  
- `mkdir -p ~/.ssh` ensures the directory exists.  
- `cat >> ~/.ssh/authorized_keys` appends the key to the file.

#### **Verify the Key on the Server:**
```bash
ssh user@target-ip "cat ~/.ssh/authorized_keys"
```
🔹 If the key is correctly added, it will be displayed in the output.

---

## **🛠 Step 3: Connect to the Server Using SSH Keys**
Now, log in **without entering a password**:

```bash
ssh user@target-ip
```

✅ If successful, this means SSH key authentication is working!

---

## **🛠 Step 4: Secure the SSH Server (Optional)**
To **disable password authentication** for security, modify the SSH config on the server:

```bash
sudo nano /etc/ssh/sshd_config
```

Change or add these lines:

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH service:

```bash
sudo systemctl restart ssh
```

🚀 **Now, only key-based authentication is allowed!**

---

## **📜 Capture the Flag (CTF) Challenges**
After setting up SSH key authentication, one can solve the following challenges:

### **🔎 Challenge 1: Locate the SSH Keys on the System**
> What command can you use to list all SSH keys on a Linux system?

### **🔐 Challenge 2: Break into a Locked User Account**
> If a user named `hacker01` cannot log in using SSH due to a permission issue, what command would you use on the server to fix SSH key file permissions?

---

### **💡 Hint for Challenge 2:**
Run:

```bash
ls -l ~/.ssh/authorized_keys
```
Check if the permissions are too open and fix them:

```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh/
```

---

## **📝 Conclusion**
Students have now learned:  
✅ How SSH public key authentication works.  
✅ How to generate and use SSH keys.  
✅ How to troubleshoot SSH authentication.  

🎯 **Now, try the CTF challenges and test your knowledge!** 💻🔥
