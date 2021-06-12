#### Encryption settings
Encryption configuration file is located at `settings/configuration/encryption.php`. We can define list of encyption configuration to be used within application. This setting also contains default encryption which should be used.

##### List
Inside `config` section we can define list of encryption configurations. This should be list of array where key represents configuration name. Each config must have following settings:

| Name | Description |
|-----|-----|
| key | Encryption key |
| cipher | This should be cipher name which you can using `openssl_get_cipher_methods` method. You can use any of it. |

##### Define default
Using `default` setting we can define default encryption. This should be config name.

##### How to use

Using `getEncrypter` method of facace class `Nishchay` we can get `Nishchay\Security\Encrypt\Encrypt` instnace. This class have two method using which we encrypt and decrypt value.

Method `getEncrypter` have only one optional parameter which should be config name of encryption. If we don't pass any config name then default encryption will be returned.

Class `Nishchay\Security\Encrypt\Encrypt` have two methods which are listed below:

| Method | Description |
|-----|-----|
| encrypt | Accepts only one argument which needs to encrypted. |
| decrypt | Pass encrypted string. If string can not be decrypted then it will return `null` |