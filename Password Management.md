# Password Management

## 1.Password Policies

**Password policy standards**

Due to compliance requirements and best practices, many companies follow established [IT security standards](https://en.wikipedia.org/wiki/IT_security_standards). While adhering to these standards does not guarantee complete security, it is a widely accepted industry practice that defines a baseline for security controls with an organization. However, compliance alone should not be the sole measure of an organization's security controls.

Some security standards include sections on password policies or guidelines. Here are a few of the most common:

* [NIST SP800-63B](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf)
* [CIS Password Policy Guide](https://www.cisecurity.org/insights/white-papers/cis-password-policy-guide)
* [PCI DSS](https://www.pcisecuritystandards.org/document_library?category=pcidss\&document=pci_dss)

**Sample password policy**

To illustrate important considerations, here is a sample password policy. It requires that all passwords:

* Minimum of 8 characters.
* Include uppercase and lowercase letters.
* Include at least one number.
* Include at least one special character.
* It should not be the username.
* It should be changed every 60 days.

***

Based on this example, we should include certain blacklisted words in our password policies. These may include, but are not limited to:

* The company's name
* Common words associated with the company
* Names of months
* Names of seasons
* Variations on the words "welcome" and "password"
* Common and easily guessable words such as "password", "123456", and "abcde"

## Enforcing password policy

### Creating a strong password

Creating a strong password doesn't have to be difficult. Tools like [PasswordMonster](https://www.passwordmonster.com/) help evaluate the strength of passwords, while [1Password Password Generator](https://1password.com/password-generator/) can generate secure ones.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfiEYfajIjKAzNd6zeTfT%2Fimage.png?alt=media&amp;token=c57e6c53-0da8-4be4-a72a-5e7764cec93c" alt=""><figcaption></figcaption></figure>

## 2.Password Managers

A common implementation for cloud password managers involves deriving encryption keys from the master password. This approach supports [Zero-Knowledge Encryption](https://blog.cubbit.io/blog-posts/what-is-zero-knowledge-encryption), which ensures that no one, not even the service provider, can access your secured data. To illustrate this, let's examine Bitwarden's approach to password derivation:

* `Master key`: Derived from the master password using a key derivation function.
* `Master password hash`: Generated using the master password (and often the master key) to authenticate the user to the cloud service.
* `Decryption key`: Created using the master key to form a symmetric key, which is then used to decrypt vault items.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FR8QDLeMdMKwyNSAjtMy3%2Fimage.png?alt=media&amp;token=63b94c5f-6881-40ba-8da5-c7c2dd59a702" alt=""><figcaption></figcaption></figure>

Some of the most popular cloud password managers are:

* [1Password](https://1password.com/)
* [Bitwarden](https://bitwarden.com/)
* [Dashlane](https://www.dashlane.com/)
* [Keeper](https://www.keepersecurity.com/)
* [Lastpass](https://www.lastpass.com/)
* [NordPass](https://nordpass.com/)
* [RoboForm](https://www.roboform.com/)

### Local password managers

Some of the most widely used local password managers are:

* [KeePass](https://keepass.info/)
* [KWalletManager](https://apps.kde.org/kwalletmanager5/)
* [Pleasant Password Server](https://pleasantpasswords.com/)
* [Password Safe](https://pwsafe.org/)

### Features

Let's imagine we use Linux, Android, and Chrome OS. We access our applications and websites from multiple devices and want to synchronize all passwords and secure notes across them. We also need extra protection through 2FA, and our budget is $5 per month. This information can help us identify the most suitable password manager for our needs.

When choosing between a cloud or local password manager, it's important to understand the available features. [Wikipedia](https://en.wikipedia.org/wiki/List_of_password_managers) offers a helpful list of both online and offline password managers, along with their key capabilities. Here are some of the most common features:

* [2FA](https://authy.com/what-is-2fa/) support.
* Multi-platform (Android, iOS, Windows, Linux, Mac, etc.).
* Browser Extension.
* Login Autocomplete.
* Import and export capabilities.
* Password generation.

### Alternatives

Passwords are the most common form of authentication, but not the only one. As we've seen throughout this module, passwords can be compromised in many ways: cracking, guessing, shoulder surfing, and more. But what if we didn't need passwords at all? Is that even possible?

By default, most operating systems and applications are built around password based authentication. However, administrators can adopt third-party identity providers or applicationss to enhance identity protection. Some of the most common alternatives include:

* [Multi-factor Authentication (MFA)](https://en.wikipedia.org/wiki/Multi-factor_authentication)
* [FIDO2](https://fidoalliance.org/fido2/), an open authentication standard that enables passwordless logins using physical devices like [YubiKey](https://www.yubico.com/). For a broader list of devices, see [Microsoft’s supported FIDO2 providers](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless#fido2-security-key-providers).
* [One-Time Passwords (OTP)](https://en.wikipedia.org/wiki/One-time_password)
* [Time-Based One-Time Passwords (TOTP)](https://en.wikipedia.org/wiki/Time-based_one-time_password)
* [IP restrictions](https://news.gandi.net/en/2019/05/using-ip-restriction-to-help-secure-your-account)
* Device compliance enforcement via tools like [Microsoft Endpoint Manager](https://www.petervanderwoude.nl/post/tag/device-compliance/) or [Workspace ONE](https://www.loginconsultants.com/enabling-the-device-compliance-with-workspace-one-uem-authentication-policy-in-workspace-one-access)

### Going passwordless

Many companies—including [Microsoft](https://www.microsoft.com/en-us), [Auth0](https://auth0.com/), [Okta](https://www.okta.com/), and [Ping Identity](https://www.pingidentity.com/en.html)—are advocating for a [passwordless](https://en.wikipedia.org/wiki/Passwordless_authentication) future. This strategy aims to remove passwords as an authentication method altogether.

[Passwordless](https://www.pingidentity.com/en/resources/blog/posts/2021/what-does-passwordless-really-mean.html) authentication is achieved when an authentication factor other than a password is used. A password is a knowledge factor, meaning it's something a user knows. The problem with relying on a knowledge factor alone is that it's vulnerable to theft, sharing, repeat use, misuse, and other risks. Passwordless authentication ultimately means no more passwords. Instead, it relies on a possession factor (something a user has) or an inherent factor (something a user is) to verify user identity with greater assurance.

As new technology and standards evolve, we need to investigate and understand the details of their implementation to determine whether those alternatives will provide the security we need for the authentication process. You can read more about Passwordless authentication and different vendor strategies:

* [Microsoft Passwordless](https://www.microsoft.com/en-us/security/business/identity-access-management/passwordless-authentication)
* [Auth0 Passwordless](https://auth0.com/passwordless)
* [Okta Passwordless](https://www.okta.com/passwordless-authentication/)
* [PingIdentity](https://www.pingidentity.com/en/resources/blog/posts/2021/what-does-passwordless-really-mean.html)
