# User Creation Script

This script automates the process of creating users on a Linux system. It reads user information from a specified file and performs various operations such as creating users, assigning groups, setting passwords, and managing home directory permissions. The script also logs actions and stores generated passwords securely.

## Prerequisites

Ensure you have the necessary permissions to create users, groups, and modify files in the specified directories.

## Script Usage

```bash
./create_users.sh USER_FILE.txt
```
## Parameters
+ "USER_FILE.txt": A semicolon-separated file containing user information. Each line should be in the format: username;group1,group2,....
## Step-by-Step Details
### 1. Check if a File Name was Provided as an Argument
The script checks if a file name was passed as an argument. If not, it logs an error and exits.
```bash
USER_FILE="$1"
if [ -z "$USER_FILE" ]; then
    log "No user file provided. Exiting."
    exit 1
fi
```
### 2. Define Log and Password Files
The script sets the log file and password file paths.
```bash
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.txt"
```
### 3. Ensure Log and Password Files Exist
The script creates the log and password files if they don't already exist and sets the appropriate permissions.
```bash
touch "$LOG_FILE"
mkdir -p "$(dirname "$PASSWORD_FILE")"
touch "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"
```
### 4. Logging Function
The script defines a logging function to log messages with timestamps.
```bash
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}
```
### 5. Check if the Employee File Exists
The script checks if the employee file exists. If not, it logs an error and exits.
```bash
if [[ ! -f "$EMPLOYEE_FILE" ]]; then
    log "Employee file $EMPLOYEE_FILE not found."
    exit 1
fi
```
### 6. Process Each Line in the Employee File
The script processes each line in the employee file. Each line contains a username and groups separated by a semicolon.
```bash
while IFS=';' read -r username groups; do
```
### 7. Check if User Already Exists
The script checks if the user already exists. If so, it logs a message and skips to the next user.
```bash
if id "$username" &>/dev/null; then
    log "User $username already exists. Skipping."
    continue
fi
```
### 8. Create User with Home Directory
The script creates the user with a home directory. If user creation fails, it logs an error and skips to the next user.
```bash
useradd -m -s /bin/bash "$username"
if [[ $? -ne 0 ]]; then
    log "Failed to create user $username."
    continue
fi
log "User $username created."
```
### 9. Set Up Primary Group
The script sets up the primary group for the user. If the group does not exist, it creates the group.
```bash
if [[ ! $(getent group "$username") ]]; then
    groupadd "$username"
    log "Group $username created."
fi
usermod -g "$username" "$username"
log "User $username assigned to group $username."
```
### 10. Set Up Additional Groups
The script assigns the user to additional groups specified in the employee file. If a group does not exist, it creates the group.

```bash
IFS=',' read -ra group_array <<< "$groups"
for group in "${group_array[@]}"; do
    if [[ ! $(getent group "$group") ]]; then
        groupadd "$group"
        log "Group $group created."
    fi
    usermod -aG "$group" "$username"
    log "User $username added to group $group."
done
```
### 11. Generate Random Password
The script generates a random password for the user and sets it. If password setting fails, it logs an error and skips to the next user.

```bash
password=$(openssl rand -base64 12)
echo "$username:$password" | chpasswd
if [[ $? -ne 0 ]]; then
    log "Failed to set password for user $username."
    continue
fi
log "Password set for user $username."
```
### 12. Store Password Securely
The script stores the generated password securely in the password file.

```bash
echo "$username,$password" >> "$PASSWORD_FILE"
log "Password stored securely for user $username."
```
### 13. Set Home Directory Permissions
The script sets the home directory permissions for the user.

```bash
chmod 700 "/home/$username"
chown "$username:$username" "/home/$username"
log "Home directory permissions set for user $username."
```
### 14. Completion Log
The script logs a message indicating that the user creation process is completed.

```bash
log "User creation process completed."
```
### Example
Create an USER_FILE.txt with the following content:
```
john;admins,developers
jane;developers
```
### Run the script:

```bash
sudo ./create_users.sh USER_FILE.txt
```
This will create users john and jane, assign them to the specified groups, generate random passwords, and store the passwords securely.

### Logging
All actions performed by the script are logged to:
 ```
 /var/log/user_management.log.
 ```

### Password Storage
Generated passwords are stored securely in 
```
/var/secure/user_passwords.txt
```
 with restricted permissions (600).

### Notes
Ensure the script has execute permissions: 
```
chmod +x create_users.sh
```
Run the script with appropriate privileges (e.g., using sudo if necessary).






