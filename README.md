# PAM Module for Authentication with PrivacyIDEA
## [Releases](https://github.com/privacyidea/privacyidea-pam/releases)
## Features
* OTP Token
* Challenge-Response Token, incl. PUSH
* Offline with HOTP Token
* Multi-Challenge

## Build
This project requires the [very good JSON parser from nlohmann](https://github.com/nlohmann/json). Put the `json.hpp` file from the `single_include` folder in `include`.

It also requires the following libraries: `libcurl4-openssl-dev`, `libssl-dev`, `libpam0g-dev`
Compilation requires g++ and make. Installation also uses strip to remove debug symbols from the module.

Compile with:

    make

Install and remove with:

    make install
	make uninstall

This will install the PAM module in `/lib/security` or `/lib64/security`

## Configuration
The following values (case-sensitive!) can be appended to the pam config file line that references this module:
| Name     | Description |
|:--------:|:----------------|
|url=|Required. URL of privacyIDEA.|
|nossl|Disable SSL certificate check. DO NOT USE IN PRODUCTION!|
|realm=|Specify the privacyIDEA realm.|
|sendEmptyPass|Sends the username and an empty pass to privacyidea prior to asking for OTP. Can be used to trigger challenges.|
|sendPassword|Sends the username and the password that is already present in the PAM stack to privacyidea prior to asking for OTP. If no password is present, the user will be prompted to enter one. Can be used to trigger challenges. Takes precedence over `sendEmptyPass`.|
|offlineFile=|Set the path to the offline file. (default is /etc/privacyidea/pam.txt).|
|pollTime=|Set the time in seconds to poll for successful push auth. Default is 0, meaning only once. Polls twice per second.|
|prompt=|Set the default prompt text for the OTP. Note: If you want to use spaces in your text, use [] like [prompt=Text with spaces].|
|debug|Enable debug logging.|
|forwardPass|Treats the entered pass as password+otp, verifies the OTP before sending the password to the next PAM module. Useful in situations where extra input cannot be requested (e.g. when logging in via XRDP). Assumes that the OTP length is 6.|

### Example

It is a good idea to use the include mechanisms of the PAM stack. This way you can
define a file `/etc/pam.d/privacyidea-auth` and replace the `@include common-auth`
in all PAM service configurations you wish to.

A sample is located in the sample folder. You may need to adapt the sample
according to the configuration design of your Linux distribution.

### Notes
#### Push behavior
If only push and **no** OTP token were triggered, the module will poll for the configured time without prompting the user for input.

If both push and OTP token were triggered, the module will prompt for the OTP and poll **once** after the user presses enter. The user can press enter with empty input to use push, just make sure the authentication was already confirmed on the smartphone.

#### SSH
Set `ChallengeResponseAuthentication yes` in `/etc/ssh/sshd_config` (or similar).

#### Centos 9
Centos 9 contains an additional config file for ssh at `/etc/ssh/sshd_config.d/50-redhat.conf` which explicitly sets `ChallengeResponseAuthentication no`, so you might want to check that to change it.
