# Step-by-Step Practical Implementation of Kerberos Authentication on Linux

## 1. Install Kerberos Packages
First, install the necessary Kerberos packages:

```bash
sudo apt update && sudo apt install krb5-user krb5-kdc krb5-admin-server -y  # Ubuntu/Debian  
```

During installation, you'll be prompted to configure the **Kerberos realm** (e.g., `EXAMPLE.COM`).

---

## 2. Configure Kerberos Files
Edit the **Kerberos configuration file** (`/etc/krb5.conf`) to define the realm and KDC.

```bash
sudo nano /etc/krb5.conf
```

Modify the following sections:

```ini
[libdefaults]
    default_realm = EXAMPLE.COM
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    forwardable = true

[realms]
    EXAMPLE.COM = {
        kdc = kerberos.example.com
        admin_server = kerberos.example.com
    }

[domain_realm]
    .example.com = EXAMPLE.COM
    example.com = EXAMPLE.COM
```
Save and exit (`Ctrl+X`, then `Y`, then `Enter`).

---

## 3. Initialize the Kerberos Database
Run the following command to create the Kerberos database:

```bash
sudo kdb5_util create -s
```

You'll be prompted to enter a **Kerberos master password**—choose a strong one and remember it.

---

## 4. Start and Enable Kerberos Services
Enable and start the **Kerberos Key Distribution Center (KDC)** and **admin server**:

```bash
sudo systemctl enable krb5-kdc
sudo systemctl start krb5-kdc

sudo systemctl enable krb5-admin-server
sudo systemctl start krb5-admin-server
```

---

## 5. Create a Kerberos Principal (User Account)
To add a user (`alice`), run:

```bash
sudo kadmin.local -q "addprinc alice"
```
You'll be prompted to set a password for **alice**.

---

## 6. Verify Initial Kerberos Authentication

### Step 1: User Enters Credentials
```bash
kinit alice
```
- The user (`alice`) enters their password.  
- The **workstation encrypts a timestamp** using the user’s password.

### Step 2: Retrieve and Display Ticket
After successful authentication, verify the Ticket Granting Ticket (TGT):

```bash
klist
```
Output example:
```
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: alice@EXAMPLE.COM

Valid starting     Expires            Service principal
03/17/25 10:00:00  03/17/25 20:00:00  krbtgt/EXAMPLE.COM@EXAMPLE.COM
```
This confirms that `alice` has received a **TGT** from the **Authentication Server (AS)**.

---

## 7. Access a Kerberized Service
If you have a service (e.g., SSH configured for Kerberos), request a **Service Ticket** from the **Ticket Granting Server (TGS)**:

```bash
ssh -o GSSAPIAuthentication=yes alice@kerberos.example.com
```
- The **TGS issues a Service Ticket**.  
- Alice logs in **without re-entering a password**.

---

## 8. Destroy the Ticket (Logout)
To remove authentication tickets:

```bash
kdestroy
```
This clears all Kerberos credentials from the system.

---

## Key Takeaways
 **Kerberos secures authentication** by using encrypted tickets instead of passwords.  
 The **Authentication Server (AS)** verifies users and issues a **TGT**.

