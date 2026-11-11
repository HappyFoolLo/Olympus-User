# Requires ActiveDirectory Module
Import-Module ActiveDirectory

$DomainOU = "DC=olympus,DC=net"
$BaseOU   = "OU=Mount Olympus,$DomainOU"

# 1. Ensure Organizational Hierarchy Exists
Write-Host "[+] Verifying Organizational Units..." -ForegroundColor Green
if (-not (Get-ADOrganizationalUnit -Filter "Name -eq 'Mount Olympus'" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit -Name "Mount Olympus" -Path $DomainOU
}
if (-not (Get-ADOrganizationalUnit -Filter "Name -eq 'Olympians (Admins)'" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit -Name "Olympians (Admins)" -Path $BaseOU
}
if (-not (Get-ADOrganizationalUnit -Filter "Name -eq 'Deities (Standard)'" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit -Name "Deities (Standard)" -Path $BaseOU
}

# 2. Define Administrative Accounts (12 Olympians - 16+ chars for strict complexity)
$Olympians = @(
    @{ Sam="zeus";       Name="Zeus";       Surname="Skyfather"; Role="DomainAdmin";     Password="Thunderbolt#2026!" },
    @{ Sam="poseidon";   Name="Poseidon";   Surname="Earthshaker"; Role="DomainAdmin"; Password="TridentDepth#2026!" },
    @{ Sam="hera";       Name="Hera";       Surname="Queen";     Role="DomainAdmin";     Password="PeacockCrown!2026!" },
    @{ Sam="demeter";    Name="Demeter";    Surname="Harvest";   Role="ServerAdmin";     Password="HarvestGrain#2026!" },
    @{ Sam="athena";     Name="Athena";     Surname="Wisdom";    Role="SecurityAdmin"; Password="ParthenonShield!2026" },
    @{ Sam="apollo";     Name="Apollo";     Surname="Sun";       Role="SecurityAdmin"; Password="GoldenLyre#2026!!" },
    @{ Sam="artemis";    Name="Artemis";    Surname="Hunt";      Role="SecurityAdmin"; Password="SilverArrow!2026!!" },
    @{ Sam="ares";       Name="Ares";       Surname="War";       Role="HelpdeskAdmin"; Password="VanguardSpear#2026!" },
    @{ Sam="aphrodite";  Name="Aphrodite";  Surname="Beauty";    Role="HelpdeskAdmin"; Password="GoldenApple!2026!!" },
    @{ Sam="hephaestus"; Name="Hephaestus"; Surname="Forge";     Role="ServerAdmin";     Password="AnvilHammer#2026!!" },
    @{ Sam="hermes";     Name="Hermes";     Surname="Messenger"; Role="HelpdeskAdmin"; Password="WingedSandals#2026!" },
    @{ Sam="hestia";     Name="Hestia";     Surname="Hearth";    Role="ServerAdmin";     Password="EternalFlame#2026!" }
)

# 3. Define Standard Accounts
$StandardGods = @(
    @{ Sam="persephone"; Name="Persephone"; Surname="Spring";      Password="Demeter01!Underworld"; EmailDomain="hades.underworld" },
    @{ Sam="dionysus";   Name="Dionysus";   Surname="Festivities"; Password="GrapeVine#2026!";      EmailDomain="olympus.net" },
    @{ Sam="nike";       Name="Nike";       Surname="Victory";     Password="WingedVictory!2026";  EmailDomain="olympus.net" },
    @{ Sam="hecate";     Name="Hecate";     Surname="Magic";       Password="CrossroadsKey!2026";  EmailDomain="olympus.net" },
    @{ Sam="pan";        Name="Pan";        Surname="Wild";        Password="FluteMelody#2026!";   EmailDomain="olympus.net" },
    @{ Sam="iris";       Name="Iris";       Surname="Rainbow";     Password="RainbowBridge!2026";  EmailDomain="olympus.net" },
    @{ Sam="nemesis";    Name="Nemesis";    Surname="Retribution"; Password="ScalesOfJustice#2026";EmailDomain="olympus.net" },
    @{ Sam="eros";       Name="Eros";       Surname="Desire";      Password="GoldenArrowhead!2026";EmailDomain="olympus.net" },
    @{ Sam="hebe";       Name="Hebe";       Surname="Youth";       Password="NectarAmbrosia#2026";EmailDomain="olympus.net" },
    @{ Sam="tyche";      Name="Tyche";      Surname="Fortune";     Password="WheelOfLuck#2026!";   EmailDomain="olympus.net" },
    @{ Sam="morpheus";   Name="Morpheus";   Surname="Dreams";      Password="ShapeShifter#2026!";  EmailDomain="olympus.net" },
    @{ Sam="triton";     Name="Triton";     Surname="DeepSea";     Password="ConchShellSound#2026";EmailDomain="olympus.net" }
)

# 4. Provision Administrative Accounts
Write-Host "`n[+] Provisioning Olympian Administrators..." -ForegroundColor Green
$AdminOU = "OU=Olympians (Admins),$BaseOU"

foreach ($god in $Olympians) {
    if (-not $god.Sam -or -not $god.Name) { continue }

    $FullName = "$($god.Name) $($god.Surname)"
    $UPN      = "$($god.Sam)@olympus.net"
    $Email    = "$($god.Sam)@olympus.net"
    $SecurePassword = ConvertTo-SecureString $god.Password -AsPlainText -Force

    if (-not (Get-ADUser -Filter "SamAccountName -eq '$($god.Sam)'" -ErrorAction SilentlyContinue)) {
        New-ADUser -Name $FullName `
                   -SamAccountName $god.Sam `
                   -GivenName $god.Name `
                   -Surname $god.Surname `
                   -DisplayName $FullName `
                   -UserPrincipalName $UPN `
                   -EmailAddress $Email `
                   -Path $AdminOU `
                   -AccountPassword $SecurePassword `
                   -Enabled $true `
                   -ChangePasswordAtLogon $false

        switch ($god.Role) {
            "DomainAdmin"   { Add-ADGroupMember -Identity "Domain Admins" -Members $god.Sam }
            "ServerAdmin"   { Add-ADGroupMember -Identity "Server Operators" -Members $god.Sam }
            "SecurityAdmin" { Add-ADGroupMember -Identity "Account Operators" -Members $god.Sam }
            "HelpdeskAdmin" { Add-ADGroupMember -Identity "Print Operators" -Members $god.Sam }
        }
        Write-Host "Created Admin: $FullName [$($god.Role)]" -ForegroundColor Cyan
    } else {
        Write-Host "User $($god.Sam) already exists. Skipping." -ForegroundColor Yellow
    }
}

# 5. Provision Standard Accounts
Write-Host "`n[+] Provisioning Standard Deities..." -ForegroundColor Green
$StdOU = "OU=Deities (Standard),$BaseOU"

foreach ($god in $StandardGods) {
    if (-not $god.Sam -or -not $god.Name) { continue }

    $FullName = "$($god.Name) $($god.Surname)"
    $UPN      = "$($god.Sam)@olympus.net"
    $Email    = "$($god.Sam)@$($god.EmailDomain)"
    $SecurePassword = ConvertTo-SecureString $god.Password -AsPlainText -Force

    if (-not (Get-ADUser -Filter "SamAccountName -eq '$($god.Sam)'" -ErrorAction SilentlyContinue)) {
        New-ADUser -Name $FullName `
                   -SamAccountName $god.Sam `
                   -GivenName $god.Name `
                   -Surname $god.Surname `
                   -DisplayName $FullName `
                   -UserPrincipalName $UPN `
                   -EmailAddress $Email `
                   -Path $StdOU `
                   -AccountPassword $SecurePassword `
                   -Enabled $true `
                   -ChangePasswordAtLogon $false

        Write-Host "Created Standard User: $FullName [Email: $Email]" -ForegroundColor Yellow
    } else {
        Write-Host "User $($god.Sam) already exists. Skipping." -ForegroundColor Yellow
    }
}

Write-Host "`n[+] Active Directory provisioning complete!" -ForegroundColor Green