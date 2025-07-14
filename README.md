# Crypto2025Zia
This is for ICT 18th batch students. 

Cipher Modes হল ক্রিপ্টোগ্রাফিতে ব্যবহৃত পদ্ধতি যা block cipher অ্যালগরিদমের এনক্রিপশন প্রক্রিয়ার গুণমান এবং নিরাপত্তা বাড়াতে ব্যবহৃত হয়। যখন ব্লক সাইজে ডেটা এনক্রিপ্ট করা হয়, তখন একই key এবং block cipher algorithm ব্যবহার করে একাধিক ব্লক এনক্রিপ্ট করার জন্য বিভিন্ন modes of operation ব্যবহার করা হয়।

এই modes-এর মধ্যে সবচেয়ে সাধারণগুলি হল ECB, CBC, CFB, এবং OFB, যা block cipher অ্যালগরিদমের আচরণকে পরিবর্তন করে।

Cipher Modes এর ধারণা:
ECB (Electronic Codebook) Mode:
এটি সবচেয়ে সহজ এবং সোজা cipher mode, যেখানে প্রতিটি ব্লক আলাদাভাবে এনক্রিপ্ট করা হয়। একটি নির্দিষ্ট key এবং block cipher algorithm ব্যবহার করে প্রতিটি ব্লক আলাদা আলাদা এনক্রিপ্ট করা হয়। ECB মুডের সবচেয়ে বড় সমস্যা হল এটি একেবারে নিরাপদ নয়, কারণ একই ইনপুট ব্লক প্রতিবার একই আউটপুট ব্লক তৈরি করবে, যা pattern বা repetition সনাক্ত করতে সাহায্য করে।
Problem with ECB: যদি কিছু ডেটাতে পুনরাবৃত্তি থাকে (যেমন একই পাসওয়ার্ড বার বার ব্যবহৃত হচ্ছে), তাহলে আক্রমণকারী সহজেই সেই পুনরাবৃত্তি সনাক্ত করতে পারে, যা নিরাপত্তা বিঘ্নিত করতে পারে।
Use Case: সাধারণত ECB mode অত্যন্ত নিরাপত্তাহীন বলে এটি বড় সিকিউরিটি ব্যবস্থায় ব্যবহার করা হয় না।
Example: ECB Mode
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.CipherInputStream;
import javax.crypto.CipherOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.security.Key;

public class ECBModeExample {
    public static void main(String[] args) throws Exception {
        // Key generation
        KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
        SecretKey secretKey = keyGenerator.generateKey();

        // Cipher instance for ECB
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");

        // Example of Encryption using ECB mode
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        byte[] encrypted = cipher.doFinal("This is a test.".getBytes());
        System.out.println("Encrypted: " + new String(encrypted));

        // Example of Decryption using ECB mode
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] decrypted = cipher.doFinal(encrypted);
        System.out.println("Decrypted: " + new String(decrypted));
    }
}
Copy
CBC (Cipher Block Chaining) Mode:
CBC হল একটি উন্নত পদ্ধতি যেখানে এনক্রিপশন প্রক্রিয়া শুরু করার জন্য একটি Initialization Vector (IV) ব্যবহার করা হয়। একটি ব্লক এনক্রিপ্ট করার জন্য, প্রথমে তার সাথে আগের ব্লকের এনক্রিপ্টেড আউটপুট XOR করা হয়। এটি একাধিক ব্লক এনক্রিপশনে শক্তিশালী নিরাপত্তা দেয় কারণ একই ইনপুটের জন্য প্রতিবার আলাদা আউটপুট তৈরি হয়।
Security Advantage: যেহেতু ব্লকগুলো একে অপরের উপর নির্ভরশীল, তাই এক ব্লকের পরিবর্তন পুরো ডেটা সিস্টেমের মধ্যে পরিবর্তন আনবে, যা আক্রমণকারীদের জন্য কঠিন করে তোলে।
Use Case: এটি নিরাপদ ব্লক সাইফার এনক্রিপশনের জন্য ব্যবহৃত হয়, বিশেষ করে যেখানে অনেক সিকিউরিটি প্রয়োজন।
Example: CBC Mode
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.CipherInputStream;
import javax.crypto.CipherOutputStream;
import javax.crypto.spec.IvParameterSpec;

public class CBCModeExample {
    public static void main(String[] args) throws Exception {
        // Key and IV generation
        KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
        SecretKey secretKey = keyGenerator.generateKey();
        byte[] iv = new byte[16]; // Initialization Vector
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);

        // Cipher instance for CBC
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");

        // Encryption using CBC mode
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec);
        byte[] encrypted = cipher.doFinal("This is a test.".getBytes());
        System.out.println("Encrypted (CBC): " + new String(encrypted));

        // Decryption using CBC mode
        cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec);
        byte[] decrypted = cipher.doFinal(encrypted);
        System.out.println("Decrypted (CBC): " + new String(decrypted));
    }
}
Copy
CFB (Cipher Feedback) Mode:
CFB হল একটি ব্লক সাইফার কিপিং মেথড যেখানে প্রতিটি ব্লক এনক্রিপ্ট করার জন্য আগের ব্লক থেকে আউটপুট ব্যবহার করা হয়। CFB বিভিন্ন ভ্যারিয়েন্টে আসে (CFB-1, CFB-8, CFB-128) যেখানে ব্লকের সাইজ পরিবর্তিত হয়। এটি stream cipher এর মতো কাজ করে, অর্থাৎ ইনপুট ডেটা ব্লক আকারে আছেঠেনা, বরং বিট বা ছোট ছোট অংশে এনক্রিপ্ট করা হয়।
Security Advantage: এটি stream cipher এর মতো কাজ করার কারণে, key এবং IV প্রক্রিয়া ব্যবহার করার মাধ্যমে সিকিউরিটি বৃদ্ধি পায়।
Use Case: এটি বিশেষত টাইম-সেন্সিটিভ ডেটা এনক্রিপশন এবং সিকিউরিটি অ্যাপ্লিকেশনগুলিতে ব্যবহৃত হয়।
Example: CFB Mode
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.IvParameterSpec;

public class CFBModeExample {
    public static void main(String[] args) throws Exception {
        // Key and IV generation
        KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
        SecretKey secretKey = keyGenerator.generateKey();
        byte[] iv = new byte[16]; // Initialization Vector
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);

        // Cipher instance for CFB
        Cipher cipher = Cipher.getInstance("AES/CFB8/PKCS5Padding");

        // Encryption using CFB mode
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec);
        byte[] encrypted = cipher.doFinal("This is a test.".getBytes());
        System.out.println("Encrypted (CFB): " + new String(encrypted));

        // Decryption using CFB mode
        cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec);
        byte[] decrypted = cipher.doFinal(encrypted);
        System.out.println("Decrypted (CFB): " + new String(decrypted));
    }
}
Copy
OFB (Output Feedback) Mode:
OFB একটি ব্লক সাইফার অপারেশন মুড যা ব্লক সাইফার এনক্রিপ্টের আউটপুটকে পরবর্তী ব্লক এনক্রিপ্ট করার জন্য ইনপুট হিসাবে ব্যবহার করে। এর মধ্যে initialization vector (IV) ব্যবহৃত হয়, যা আগে এনক্রিপ্ট করা আউটপুটের সাথে XOR করা হয়। এই মোডটি CFB এর মতো কাজ করে, তবে এতে ইনপুট ডেটা এবং সিকোয়েন্সের এনক্রিপ্টেড আউটপুটের মধ্যে সম্পর্ক ভিন্ন।
Security Advantage: এটি নিরাপদ এবং সিকিউরিটি বৃদ্ধি পায় কারণ এতে আগের ব্লকগুলোর প্রভাব নেই এবং এটি সহজে error propagation রোধ করতে সাহায্য করে।
Use Case: এটি এমন অ্যাপ্লিকেশনে ব্যবহৃত হয় যেখানে ছোট ব্লকগুলির এনক্রিপশন প্রয়োজন এবং ফাইল বা স্ট্রিম প্রক্রিয়াকরণে কাজ আসে।
Example: OFB Mode
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.IvParameterSpec;

public class OFBModeExample {
    public static void main(String[] args) throws Exception {
        // Key and IV generation
        KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
        SecretKey secretKey = keyGenerator.generateKey();
        byte[] iv = new byte[16]; // Initialization Vector
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);

        // Cipher instance for OFB
        Cipher cipher = Cipher.getInstance("AES/OFB/PKCS5Padding");

        // Encryption using OFB mode
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec);
        byte[] encrypted = cipher.doFinal("This is a test.".getBytes());
        System.out.println("Encrypted (OFB): " + new String(encrypted));

        // Decryption using OFB mode
        cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec);
        byte[] decrypted = cipher.doFinal(encrypted);
        System.out.println("Decrypted (OFB): " + new String(decrypted));
    }
}
Copy
Cipher Modes (ECB, CBC, CFB, OFB) হল ব্লক সাইফার এনক্রিপশনকে আরও শক্তিশালী এবং নিরাপদ করার জন্য ব্যবহৃত প্রক্রিয়া।

ECB সহজ হলেও নিরাপত্তাহীন।
CBC, CFB, এবং OFB সিকিউর এবং সাধারণত নিরাপত্তা অ্যাপ্লিকেশনগুলিতে ব্যবহৃত হয়।


Copied from SATT ACADEMY. Visit us at: https://sattacademy.com
