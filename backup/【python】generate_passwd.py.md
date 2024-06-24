```python
import random
import string
import sys


def generate_password(length=16):
    if length < 8:
        raise ValueError("密码长度不能少于8位")

    all_chars = string.ascii_letters + string.digits + string.punctuation

    password = [
        random.choice(string.ascii_lowercase),
        random.choice(string.ascii_uppercase),
        random.choice(string.digits),
        random.choice(string.punctuation)
    ]

    password += random.choices(all_chars, k=length - 4)

    random.shuffle(password)

    return ''.join(password)


if __name__ == "__main__":
    print(len(sys.argv))
    if len(sys.argv) > 1:
        try:
            length = int(sys.argv[1])
        except ValueError:
            print("请输入一个有效的整数作为密码长度")
            sys.exit(1)
    else:
        length = 16

    try:
        print(generate_password(length))
    except ValueError as e:
        print(e)
        sys.exit(1)
```