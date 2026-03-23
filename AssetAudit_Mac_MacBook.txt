#!/bin/bash
# Check if mutt configuration already exists and delete it
if [ -f ~/.muttrc ]; then
    
    rm ~/.muttrc
fi

# Function to install and configure mutt
install_and_configure_mutt() {
    # Check if Homebrew is installed
    if ! command -v brew &> /dev/null; then
        echo "Homebrew is not installed. Installing Homebrew..."
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        
        # Add Homebrew to PATH
        {
            echo '# Homebrew'
            echo 'eval "$(/opt/homebrew/bin/brew shellenv)"'
        } >> ~/.zprofile
        
        echo "Homebrew has been installed. Please run the following command to add it to your PATH:"
        echo 'eval "$(/opt/homebrew/bin/brew shellenv)"'
        echo "Then, restart your terminal or run 'source ~/.zprofile' or 'source ~/.bash_profile' to apply the changes."
        return
    fi 

    # Install mutt if it's not installed
    if ! command -v mutt &> /dev/null; then 
        
        brew install mutt 
    fi

    # Check if mutt was installed successfully 
    if ! command -v mutt &> /dev/null; then
        echo "Mutt installation failed. Please ensure Homebrew is in your PATH."
        echo "Run 'eval \"\$(/opt/homebrew/bin/brew shellenv)\"' to temporarily add it."
        echo "Then, restart your terminal or source your profile file."
        return
    fi

    # Create mutt configuration if it doesn't exist
    if [ ! -f ~/.muttrc ]; then
        
        cat <<EOL > ~/.muttrc
set smtp_url = "smtp://it.assetaudit@host:smtp.host:port no/"
set from = "it.assetaudit@host"
set realname = "it.assetaudit@host"
set ssl_starttls = yes
set ssl_force_tls = yes
set smtp_authenticators = "login"
set smtp_pass = "iFG&4na7"  # Consider using an app password
EOL

echo "Mutt configuration created."  
    else
      echo "Mutt configuration already exists." 
    fi
      
}

# Call the function to install and configure mutt
install_and_configure_mutt

validate_email() {
    local email="$1"
    local pattern="^([a-zA-Z0-9._%+-]+@host
    
    
    [[ $email =~ $pattern ]]
}


send_verification_email() {
    local recipientEmail="$1"
    local verificationCode="$2"
    local fullName="$3"

    

    emailBody=$(cat <<EOF
<html>
    <head>
        <title>Verification Code for IT Audit</title>
    </head>
    <body>
        <p>Dear $fullName,</p>
        <p>As part of our ongoing IT audit, we require verification of your email address. Please enter the code below to confirm your identity and complete the process:</p>
        
        <h3>Your Verification Code: $verificationCode</h3>
        
        <p>If you have any concerns, kindly reach out to abdullah.lafir@alibaba-inc.com</p>
        
        <br />
        <p>Best Regards,</p>
        <p>IT LK</p>
    </body>
</html>
EOF
)

    # Send the email using mutt
    echo -e "Subject: IT Audit | Email Verification\nContent-Type: text/html\n\n$emailBody" | mutt -e "set content_type=text/html" -s "IT Audit | Email Verification" -- "$recipientEmail"
}

# Define colors
RED='\033[0;31m'
GREEN='\033[0;32m' 
YELLOW='\033[0;33m'
NC='\033[0m' # No Color


# Function to get and validate user input
# Function to get and validate user input
get_validated_user_input() {
    while true; do
        read -p "Enter the Job Number (e.g., WB123456 or 123456): " jobNumber
        # Convert the job number to uppercase
        userJobNumber=$(echo "$jobNumber" | tr '[:lower:]' '[:upper:]')

        # Validate the job number (allow both cases for WB and a 6-digit number)
        if [[ "$userJobNumber" =~ ^(WB[0-9]{6,8})$|^([0-9]{6})$ ]]; then
            break
        else
            echo -e "${RED}ERROR: INVALID JOB NUMBER, PLEASE ENTER A VALID JOB NUMBER!${NC}"
            echo ""  # Add a blank line for spacing
        fi
    done


     echo ""  # Add a blank line for spacing

    while true; do
        read -p "Enter the Country (LK, PK, NP, BD, MM): " country
        userCountry=$(echo "$country" | tr '[:lower:]' '[:upper:]')
        if [[ "$userCountry" =~ ^(LK|PK|NP|BD|MM)$ ]]; then
            
            break
        else
            echo -e "${RED}ERROR: INVALID COUNTRY, PLEASE ENTER A VALID COUNTRY CODE!${NC}"
            echo ""  # Add a blank line for spacing
        fi
    done
echo ""  # Add a blank line for spacing
    while true; do
        read -p "Is this a Laptop or Desktop? Enter 'L' for Laptop or 'D' for Desktop: " deviceType
        deviceType=$(echo "$deviceType" | tr '[:lower:]' '[:upper:]')
        if [[ "$deviceType" =~ ^(L|D)$ ]]; then
            deviceType="MACBOOK"
            [[ "$deviceType" == "D" ]] && deviceType="DESKTOP"
           
            break
        else
            echo -e "${RED}ERROR: INVALID DEVICE TYPE. PLEASE ENTER 'L' FOR LAPTOP OR 'D' FOR DESKTOP!${NC}"
            echo ""  # Add a blank line for spacing
        fi
    done

    echo ""  # Add a blank line for spacing  

    while true; do
        read -p "Is this device currently in Use or from the IT store? Enter 'IU' for In Use or 'IS' for IT Store: " inventoryType
        inventoryType=$(echo "$inventoryType" | tr '[:lower:]' '[:upper:]')
        if [[ "$inventoryType" =~ ^(IU|IS)$ ]]; then
            inventoryType="In Use - Assigned to Employee"
            [[ "$inventoryType" == "IS" ]] && inventoryType="In Store - Not Assigned"
            
            break
        else
            echo -e "${RED}ERROR: INVALID INVENTRY TYPE. PLEASE ENTER 'IU' FOR IN USE OR 'IS' FOR IT Store!${NC}"
            echo ""  # Add a blank line for spacing
        fi
    done

    echo ""  # Add a blank line for spacing

    while true; do
        read -p "Do you belong to Headquarters, Sort Center, Warehouse, or Hub? Enter 'HQ' for Headquarters, 'S' for Sort Center, 'W' for Warehouse, or 'H' for Hub: " belonging
        belonging=$(echo "$belonging" | tr '[:lower:]' '[:upper:]')
        case "$belonging" in
            HQ) belonging="Headquarters" ;;
            S) belonging="Sort Center" ;;
            W) belonging="Warehouse" ;;
            H) belonging="Hub" ;;
            *) echo -e "${RED}ERROR: INVALID SELECTION. PLEASE ENTER 'HQ', 'S', 'W', or 'H'.${NC}" && continue ;;
            

        esac 
        echo ""  # Add a blank line for spacing
       
        break
    done

    echo ""  # Add a blank line for spacing

    read -p "Which $belonging office location do you belong to? " location


    # Get the Manufacturer and Model information
    systemInfo=$(system_profiler SPHardwareDataType | awk '/Model Name|Model Identifier|Manufacturer/ {print $0}')

    # Initialize VMStatus
    VMStatus="Not VM"

    #!/bin/bash

# Define the list of companies
CompanyList=(
    "1) Sister Company1"
    "2) Sister Company2"
    "3) Sister Company3"
    "4) Sister Company4"
    "5) Sister Company5"
)

# Display the company list
echo ""
echo "Select the Company by number:"
for company in "${CompanyList[@]}"; do
    echo "$company"
done

# Start loop to keep asking for valid input
while true; do
    read -p "ENTER THE NUMBER CORRESPONDING TO THE COMPANY: " mainSelection
    
    # Validate input to ensure it is a number between 1 and 5
    if [[ "$mainSelection" =~ ^[1-5]$ ]]; then
        # Extract the company name by removing the number and the parentheses
        Company=$(echo "${CompanyList[$((mainSelection - 1))]}" | sed 's/^[0-9]*) //')
        break
    else
        echo -e "\033[31mERROR: INVALID SELECTION. PLEASE ENTER A NUMBER BETWEEN 1 TO 5.\033[0m"
    fi
done
 
# Output the selected company name

# Ask for the Auditor's Full Name and format it
read -p "Enter the Full Name of the Auditor: " AuditorfullName

# Format the name: capitalize the first letter of each word
formattedFullName=$(echo "$AuditorfullName" | awk '{for(i=1;i<=NF;i++) $i=toupper(substr($i,1,1)) tolower(substr($i,2));}1')

# Output the formatted full name
echo "Formatted Full Name: $formattedFullName"



    # Check if the system is a VM based on known virtualization manufacturers or models
    if echo "$systemInfo" | grep -q -E "VMware|Parallels|VirtualBox|QEMU|KVM"; then
        VMStatus="VM"
    fi

echo ""  # Add a blank line for spacing
    # All inputs have been validated
    echo -e "${RED}PLEASE WAIT TILL YOUR INFORMATION IS PROCESSED!${NC}"
    echo ""  # Add a blank line for spacing
}


# Function to audit laptop info
audit_laptop_info() {
    endUserEmail="$1"



    laptopModel=$(system_profiler SPHardwareDataType | awk '/Model Identifier/{print $3}')
    laptopbrand=$(system_profiler SPHardwareDataType | awk '/Model Name/{for (i=3; i<=NF; i++) printf $i " "; print ""}')

    totalMemoryBytes=$(sysctl -n hw.memsize)
    ramGB=$(echo "scale=2; $totalMemoryBytes / 1024 / 1024 / 1024" | bc)

   # Get the total size of the entire disk (e.g., /dev/disk0) on macOS
hardDiskGB=$(diskutil info /dev/disk0 | grep "Disk Size" | awk '{print $3}')
echo $hardDiskGB

    

     # Get the primary storage volume name and type
    storageVolume=$(diskutil info disk0 | awk '/Volume Name/{print $3}')
    storageType=$(diskutil info disk0 | awk '/Media Type/{print $3}')
    DiskData="Storage Volume:$storageVolume | Storage Type:$storageType"

    processor=$(sysctl -n machdep.cpu.brand_string)
    generation=$(echo "$processor" | grep -o '[0-9][0-9][0-9][0-9]' | head -1)
    [ -z "$generation" ] && generation="Unknown"

    osType=$(sw_vers -productName)
    osVersion=$(sw_vers -productVersion)

    # Hard-coded main category
    endUserCategory="Computer"
    
    # Get the installation date
    installDate=$(system_profiler SPSoftwareDataType | awk '/Install Date/{print $3, $4, $5, $6, $7}')

    installDateFormatted=${installDate:-"Installation date not found."}

    # Extract system serial number
systemSerialNumber=$(system_profiler SPHardwareDataType | awk '/Serial Number/{print $4}')

# Get the graphics card (DISPCARD)
graphicsCard=$(system_profiler SPDisplaysDataType | awk '/Chipset Model/{print $3, $4}')

# Get the LAN adapter (LANADPT)
lanAdapter=$(ifconfig en0 | awk '/ether/{print $2}')

# Get the Product ID (Windows Product ID equivalent) - using a dummy product number
# Since there's no equivalent to `Win32_OperatingSystem.SerialNumber` in Linux, using /etc/os-release as an example.
productID=$(sw_vers -productVersion)

# Get the current date and time with milliseconds
dateTimeFormatted=$(date +"%Y-%m-%d %H:%M:%S").$(printf "%03d" $(($(date +%N)/1000000)))
uniqueRecordID=${userCountry}$(date +"%Y%m%d%H%M%S")$(printf "%03d" $(($(date +%N)/1000000)))


# Generate the new hostname using the retrieved employeeCountry and device serial number
serialNumber=$(system_profiler SPHardwareDataType | awk '/Serial Number/{print $4}')

# Create a unique Asset ID
uniqueAssetID="${userCountry}-${serialNumber}"  

# Get the UUID
uuid=$(system_profiler SPHardwareDataType | awk '/UUID/ {print $3}')

# Check if the system serial number is empty or a placeholder
if [[ "$systemSerialNumber" == "N/A" || -z "$systemSerialNumber" ]]; then
    echo "System serial number is 'N/A' or empty. Trying to get the physical disk serial number..."

    # Get the physical disk serial number as a fallback
    physicalDiskSerialNumber=$(diskutil info disk0 | awk '/Device Identifier/ {print $3}')
    serialNumber=${physicalDiskSerialNumber:-"Not Available"}
else
    serialNumber="$systemSerialNumber"
fi

# Get the current hostname
hostName=$(hostname)


    osGenuineStatus=""
    if ! open -Ra "App Store" &>/dev/null || softwareupdate -l | grep -q "No new software available."; then
        osGenuineStatus+="Not Genuine (Cracked)"    
    else
        osGenuineStatus+="Genuine"
    fi

    wifiSSID=$(networksetup -getairportnetwork en0 | cut -d ':' -f2 | sed 's/^ *//g')
    wifiMacAddress=$(ifconfig en0 | awk '/ether/{print $2}')
    
    macAddresses=""
    for interface in en0 en1; do
        mac=$(ifconfig "$interface" | awk '/ether/{print $2}')
        if [ -n "$mac" ]; then
            macAddresses+="MAC ($interface): $mac\n"
        fi
    done 

  #--------------------------------INSERT INTO SQL SERVER--------------------------------#

# Load SQL connection details from environment variables (ensure these are set before running the script)
sqlServer="${SQL_SERVER:-xxx.xxx.xxx.xxx,port no\\SQLxxx}"
sqlDatabase="${SQL_DATABASE:-assetaudit}"
sqlUser="${SQL_USER:-assetaudit_new}"
sqlPassword="${SQL_PASSWORD:-new pass!}"
# Daraz_AssetAudit

# Test the connection
connectionTest=$(sqlcmd -S "$sqlServer" -d "$sqlDatabase" -U "$sqlUser" -P "$sqlPassword" -Q "SELECT 1" 2>&1)


# SQL Insert Query
sqlInsertQuery="INSERT INTO LaptopAudit 
(RecordId, AuditedDateTime, ComputerModel, RAM_GB, HardDisk_GB, ProcessorType, GenerationType, InstallDate, SerialNumberInfo, GraphicsCard, LanAdapter, ProductID, WifiMacAddressInfo, OSTypeInfo, OSVersionInfo, OSGenuineStatusInfo, WifiSSIDInfo, LaptopBrandInfo, MACAddressesInfo, DiskInfo, UserEmail, AssetId, employeeJobNumber, employeeCountry, endUserSubCategory, endUserInventoryType, endUserbelonging, endUserLocation, vmStatus, hostName, endUserCategory, Company, AuditedBy)
VALUES
('$uniqueRecordID', '$dateTimeFormatted', '$laptopModel', '$ramGB', '$hardDiskGB', '$processor', '$generation', '$installDateFormatted', '$serialNumber', '$graphicsCard', '$lanAdapter', '$productID', '$wifiMacAddress', '$osType', '$osVersion', '$osGenuineStatus', '$wifiSSID', '$laptopbrand', '$macAddresses', ' $DiskData', '$endUserEmail', '$uniqueAssetID', '$jobNumber', '$userCountry', '$deviceType', '$inventoryType', '$belonging', '$location', '$VMStatus', '$hostName', 'Computer', '$Company', '$formattedFullName');"

# Run the SQL insert query using sqlcmd tool
output=$(sqlcmd -S "$sqlServer" -d "$sqlDatabase" -U "$sqlUser" -P "$sqlPassword" -Q "$sqlInsertQuery" 2>&1) 



echo ""  # Add a blank line for spacing
echo -e "${GREEN}THANKYOU!  SYSTEM EXISTING......${NC}"
echo ""  # Add a blank line for spacing

}

# Main execution starts here
attempts=0
maxAttempts=3
verified=false

while [ "$verified" = false ]; do
    if [ $attempts -ge $maxAttempts ]; then
        echo "Maximum attempts reached. Exiting..."
        exit 1
    fi

echo ""  # Add a blank line for spacing

    # Ask for recipient email
    read -p "Enter your office email address: " recipientEmail

    if validate_email "$recipientEmail"; then
        username="${recipientEmail%@*}"
        fullName=$(echo "$username" | sed 's/\./ /g' | awk '{for(i=1;i<=NF;i++)$i=toupper(substr($i,1,1)) tolower(substr($i,2))}1')

        # Generate verification code
        verificationCode=$((RANDOM % 900000 + 100000))

        # Send verification email
        send_verification_email "$recipientEmail" "$verificationCode" "$fullName"

        echo ""  # Add a blank line for spacing
        # Ask for verification code
        read -p "Enter the verification code sent to your email: " userCode

        # Check if the verification code is correct
        if [ "$userCode" == "$verificationCode" ]; then
            echo -e "${GREEN}EMAIL VERIFICATION SUCCESFULL. PROCEEDING...${NC}"
            echo ""  # Add a blank line for spacing

            endUserEmail="$recipientEmail"
            get_validated_user_input # Collect other inputs after email verification
            audit_laptop_info "$endUserEmail"
            verified=true
        else
            echo -e "${RED}ERROR: INCORRECT CODE. TRY AGAIN."
            ((attempts++))
        fi
    else
        echo -e "${RED}ERROR: INVALID EMAIL ADDRESS. PLEASE TRY AGAIN."
        ((attempts++))
    fi  

done

