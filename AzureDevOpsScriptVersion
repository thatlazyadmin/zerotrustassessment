# Ensure the script is running in PowerShell 7 or higher
if ($PSVersionTable.PSVersion.Major -lt 7) {
    Write-Host "PowerShell 7.0 or higher is required. Attempting to install PowerShell 7..."
    try {
        # Download and install PowerShell 7
        $installerUrl = "https://github.com/PowerShell/PowerShell/releases/download/v7.3.7/PowerShell-7.3.7-win-x64.msi"
        $installerPath = "$env:TEMP\PowerShell-7.msi"
        Invoke-WebRequest -Uri $installerUrl -OutFile $installerPath -UseBasicParsing
        Write-Host "Downloaded PowerShell 7 installer."

        # Install PowerShell 7 silently
        Start-Process -FilePath "msiexec.exe" -ArgumentList "/i `"$installerPath`" /quiet /norestart" -Wait
        Write-Host "PowerShell 7 installed successfully."

        # Relaunch the script in PowerShell 7
        $newPwshPath = "C:\Program Files\PowerShell\7\pwsh.exe"
        if (Test-Path $newPwshPath) {
            Write-Host "Relaunching the script in PowerShell 7..."
            Start-Process -FilePath $newPwshPath -ArgumentList "-File `"$PSCommandPath`"" -NoNewWindow
            exit
        } else {
            Write-Error "Failed to locate the newly installed PowerShell 7 executable."
            exit 1
        }
    } catch {
        Write-Error "Failed to install PowerShell 7: $_"
        exit 1
    }
} else {
    Write-Host "PowerShell 7.0 or higher is already installed."
}

# Output debugging information for environment variables (without exposing secrets)
Write-Host "Debugging Environment Variables:"
Write-Host "CLIENTID: $env:CLIENTID"
Write-Host "TENANTID: $env:TENANTID"
if ($env:CLIENTSECRET) {
    Write-Host "CLIENTSECRET is set."
} else {
    Write-Error "CLIENTSECRET is not set. Ensure the CLIENTSECRET variable is correctly configured in your pipeline."
    exit 1
}

# Retrieve environment variables for authentication
try {
    $clientSecret = ConvertTo-SecureString -AsPlainText $env:CLIENTSECRET -Force
    [pscredential]$credential = New-Object System.Management.Automation.PSCredential($env:CLIENTID, $clientSecret)
} catch {
    Write-Error "Failed to retrieve or process authentication variables. Ensure CLIENTID, CLIENTSECRET, and TENANTID are correctly configured."
    exit 1
}

# Authenticate with Microsoft Graph
try {
    Connect-MgGraph -TenantId $env:TENANTID -ClientSecretCredential $credential
    Write-Host "Authentication successful with Microsoft Graph."
} catch {
    Write-Error "Failed to authenticate with Microsoft Graph. Check your CLIENTID, CLIENTSECRET, and TENANTID values."
    exit 1
}

# Install the Zero Trust Assessment module if not already installed
try {
    if (-not (Get-Module -ListAvailable -Name ZeroTrustAssessment)) {
        Install-Module -Name ZeroTrustAssessment -AllowPrerelease -Force -AllowClobber -Scope CurrentUser
        Write-Host "Zero Trust Assessment module installed successfully."
    } else {
        Write-Host "Zero Trust Assessment module is already installed."
    }
} catch {
    Write-Error "Failed to install the Zero Trust Assessment module: $_"
    exit 1
}

# Import the Zero Trust Assessment module
try {
    Import-Module ZeroTrustAssessment
    Write-Host "Zero Trust Assessment module imported successfully."
} catch {
    Write-Error "Failed to import the Zero Trust Assessment module: $_"
    exit 1
}

# Run the Zero Trust Assessment
try {
    $results = Invoke-ZTAssessment
    Write-Host "Zero Trust Assessment executed successfully."
} catch {
    Write-Error "An error occurred while running the Zero Trust Assessment: $_"
    exit 1
}

# Save results in a CSV format for easy Excel consumption
try {
    $results | Export-Csv -Path "./ZeroTrustAssessmentResults.csv" -NoTypeInformation -Encoding UTF8
    Write-Host "Results saved to ZeroTrustAssessmentResults.csv."
} catch {
    Write-Error "Failed to save the assessment results to a CSV file: $_"
    exit 1
}
