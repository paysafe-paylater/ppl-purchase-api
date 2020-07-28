# Webhooks

When authorizing a transaction, a callbackUrl can be provided. This URL, from that point on, will receive messages every time the status of the transaction changes. Just like all API responses, the message, once decrypted, is a PurchaseOperationResponse. Before the message can be read, it must be validated and decoded. The message is AES Cipher encrypted with a 32 bytes SHA-256 hash of the default secret key. When using one of the provided SDKs, the webhook decrypt functionality can be used. Check the paragraph 'Webhooks' in the corresponding SDK documentation for more information on how to use this functionality. If using the SDKs is not an option, the following examples should give an idea on how to decrypt it:

**Java Example**
```java
import java.nio.ByteBuffer;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;

/**
 * Decrypts the contents of a webhook message.
 *
 * @param webhookMessage the encrypted JSON webhookMessage.
 * @param secretKey your default secret key (supplied by the Paysafe integration team).
 * @return the decrypted webhook JSON string.
 */
public static String decrypt(String webhookMessage, String secretKey) {
  try {
    byte[] strToDecryptBytes = Base64.getDecoder().decode(webhookMessage);
    ByteBuffer byteBuffer = ByteBuffer.wrap(strToDecryptBytes);

    Cipher cipher = getCipher(byteBuffer, secretKey);

    byte[] encryptedText = new byte[byteBuffer.remaining()];
    byteBuffer.get(encryptedText);

    return new String(cipher.doFinal(encryptedText), CHARSET);
  } catch (Exception e) {
    throw new WebhookDecryptionException("Decryption of webhook message failed.", e);
  }
}

/**
 * Creates a cipher based on the secret key and the data in the message.
 */
private static Cipher getCipher(ByteBuffer byteBuffer, String secretKey) throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, InvalidAlgorithmParameterException {
  //First byte specifies IV length (IV is 12 bytes long)
  int ivLength = byteBuffer.get();
  if (ivLength != 12) { // check input parameter
      throw new WebhookDecryptionException("invalid iv length: " + ivLength);
  }

  //Next 12 bytes are the IV
  byte[] iv = new byte[ivLength];
  byteBuffer.get(iv);

  byte[] unhashedSecretKey = secretKey.getBytes(CHARSET);
  MessageDigest sha = MessageDigest.getInstance("SHA-256");
  byte[] hashedSecretKey = sha.digest(unhashedSecretKey);

  Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
  SecretKeySpec secretKeySpec = new SecretKeySpec(hashedSecretKey, "AES");
  GCMParameterSpec ivParameterSpec = new GCMParameterSpec(128, iv);
  cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivParameterSpec);
  return cipher;
}

```

**PHP example**
```php
use function base64_decode;
use function base64_encode;
use function hash;
use function json_decode;
use function openssl_cipher_iv_length;
use function openssl_decrypt;
use function strlen;
use function substr;

use const OPENSSL_RAW_DATA;

/**
  * @param string $webhookMessage
  * @param string $paysafePlSecretKey
  * @return string
  * @throws WebhookDecrypterException
  */
protected function decryptMessage(string $webhookMessage, string $paysafePlSecretKey): string
{
    $encrypted = base64_decode($webhookMessage, true);
    if ($encrypted === false) {
        throw new WebhookDecrypterException(
            'Unable to decode webhook message. Message contains characters from outside the base64 alphabet.'
        );
    }
    if (base64_encode($encrypted) !== $webhookMessage) {
        throw new WebhookDecrypterException('Unable to decode webhook message. Message is not base64 encoded.');
    }

    $cipher = 'aes-256-gcm';
    $tagLength = 16;

    $ivLength = openssl_cipher_iv_length($cipher);
    if ($ivLength === false) {
        throw new WebhookDecrypterException('Unable to calculate iv length for cipher: ' . $cipher);
    }

    $minLength = 1 + $ivLength + $tagLength;
    if (strlen($encrypted) <= $minLength) {
        throw new WebhookDecrypterException('Unable to decode webhook message. Message to short.');
    }

    $iv = substr($encrypted, 1, $ivLength);
    $cipherText = substr($encrypted, 1 + $ivLength, -$tagLength);
    $tag = substr($encrypted, -$tagLength);
    $key = substr(hash('sha256', $paysafePlSecretKey, true), 0, 32);
    return (string) openssl_decrypt($cipherText, $cipher, $key, OPENSSL_RAW_DATA, $iv, $tag);
}
```