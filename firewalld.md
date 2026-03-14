# Firewalld and Firewall-Cmd

---

## 1. Introduction to Firewalld

`firewalld` is a dynamic firewall manager that uses **Zones** to categorize network traffic.

### Key Components

* **Zones:** Groups of rules based on the trust level of a network interface (e.g., `public`, `internal`, `trusted`).
* **Services:** Templates that map to specific ports (e.g., `ssh` = port 22).
* **Ports:** Manual entry of a port and protocol (e.g., `8080/tcp`).
* **Rich Rules:** Complex, conditional rules based on source IP, destination, and action.

---

## 2. Basic Syntax and Lifecycle

Every `firewall-cmd` command follows a standard structure.

### Command Syntax

`firewall-cmd [Options] [Zone] [Action]`

### The Two States

1. **Runtime:** Changes take effect immediately but vanish after a reboot.
2. **Permanent:** Changes are saved to XML files but require a **reload** to take effect.

**Essential Commands:**

```bash
# Check status
sudo systemctl status firewalld

# List everything in the default zone
sudo firewall-cmd --list-all

# Save and apply permanent changes
sudo firewall-cmd --reload

```

---

## 3. Working with Zones and Sources

Zones define the "perimeter." An interface or source IP belongs to exactly one zone.

### Scenarios

* **View active zones:** `sudo firewall-cmd --get-active-zones`
* **Move an interface:** `sudo firewall-cmd --permanent --zone=internal --change-interface=ens33`
* **Assign a source IP to a zone:** ```bash
# Ensure traffic from this IP always hits the 'trusted' zone



```
sudo firewall-cmd --permanent --zone=trusted --add-source=192.168.10.50

```



---

## 4. Services and Ports

This is the most common way to allow traffic.

### By Services

```bash
# Scenario: Allow web traffic on a standard server
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

```

### Ports (The Manual Way)

```bash
# Scenario: Your app runs on a non-standard port 8080
sudo firewall-cmd --permanent --add-port=8080/tcp

# Scenario: Remove the port once testing is done
sudo firewall-cmd --permanent --remove-port=8080/tcp

```

---

## 5. Advanced Logic: Rich Rules

Rich Rules allow you to combine sources, ports, and logging into a single rule.

### Scenario: Admin-Only Database Access

Allow only the **Admin VM** (`192.168.10.20`) to access the **DB Server** (`3306`), while rejecting all other IPs even if they have the database password.

```bash
# Apply on the DB Server
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.10.20" port port="3306" protocol="tcp" accept'

# Scenario: Remove a specific rich rule
sudo firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.10.20" port port="3306" protocol="tcp" accept'

```

---

## 6. Firewalld as a Router (Forwarding)

In a multi-subnet environment (e.g., Subnet 10.0 and Subnet 20.0), the "Router VM" needs to permit traffic passing **through** it.

### Step 1: Enable Kernel Forwarding

```bash
# Temporary
sudo sysctl -w net.ipv4.ip_forward=1
# Permanent
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

```

### Step 2: The Forward Rule

Standard zone rules only protect the "Router" itself. To let traffic pass through, use a **Direct Rule**.

```bash
# Scenario: Allow SSH to pass through the router to the internal network
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -p tcp --dport 22 -j ACCEPT

# Scenario: Allow established traffic to return (Stateful inspection)
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -m state --state RELATED,ESTABLISHED -j ACCEPT

```

---

## 7. Troubleshooting Checklist

| Symptom | Common Culprit | Diagnostic Command |
| --- | --- | --- |
| **Ping works, SSH fails** | Firewall allows ICMP but blocks TCP Port 22 | `firewall-cmd --list-services` |
| **No route to host** | `ip_forward` is 0 or Router is dropping packets | `tcpdump -i any port 22` |
| **Connection Refused** | App is not running or only listening on `127.0.0.1` | `ss -tulpn` |
| **Changes not working** | Applied permanently but didn't reload | `firewall-cmd --reload` |

---
