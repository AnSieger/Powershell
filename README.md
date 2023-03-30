# Powershell
Helpfull Powershell scripts


## How to hide unlicensed Mailboxes in Global Address List (GAL)
When an account loose the Exchange license, the account is still visible in the Global Address List (GAL).
With this Powershell script all users without an Exchange license will be hidden:

```
Connect-ExchangeOnline
Connect-AzureAD
```

### Get all user who are in the GAL
 
```
$GALUsers = Get-Recipient -RecipientTypeDetails UserMailbox -ResultSize Unlimited | Where-Object { $_.HiddenFromAddressListsEnabled -eq $false }
$GALUsers | Format-Table DisplayName, PrimarySmtpAddress, Alias
```

### Hide users without license or blocked:
```
# Get all user mailboxes
$AllUsers = Get-Recipient -RecipientTypeDetails UserMailbox -ResultSize Unlimited

foreach ($User in $AllUsers) {
    # Get AzureAD user information
    $AzureADUser = Get-AzureADUser -ObjectId $User.ExternalDirectoryObjectId

    # Hide users if the account has no Exchange licence or is locked out
    if (($AzureADUser.AssignedLicenses -eq $null -or $AzureADUser.AssignedLicenses.Count -eq 0) -or ($AzureADUser.AccountEnabled -eq $false)) {
        # Set the property HiddenFromAddressListsEnabled to "True" to hide the user from the GAL
        Set-Mailbox -Identity $User.PrimarySmtpAddress -HiddenFromAddressListsEnabled $true
        Write-Host "User $($User.DisplayName) hide in Global Addresslist."
    }
}
```
