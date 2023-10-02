The default ignition file create by IBM Satellite location does not include a second user that can be usefull if there is ever a need to login to the hosts if an issue arises. To add a second user copy the block below for the sigex user. 

```
{
  "ignition": {
    "version": "3.1.0"
  },
  "passwd": {
    "users": [
      {
        "name": "core",
        "sshAuthorizedKeys": [
          ""
        ]
      },
      {
        "name": "sigex",
        "passwordHash": "$2y$10$AHTNnDdsyPfBapQhrfofNuGrwVQ8OxWFbe8N3gG675Svz2bL0JiZe",
        "groups": [ "wheel", "sudo"],
        "sshAuthorizedKeys": [
          "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxoIyYZ0TE6G5mJJfrB5NpJZ6gaKWSbFigOxmoXeJQPzSTNk4Z0etDWm3p0x21ALLzyBYPHM+xR2KbKlicnV7MrAFQ3lHuIeqGOTElQmt9rPqWv5VN5JZgryYSpZR8nqjAcBmAWL2xOLqvklAXwU4y7UjamqeryP/gV5kWn5tPg9PfeoOuL74rpwf7CtRW5jqP6cyxMLzbKW5yCl1Nnv5hFWotofO+M7o+OiMjRL5PwnmhzuXegP8RqHr57vQt1MZ+1zU2eiZAubB5igHZBPFsRZIHaangO9Zp0Wcefn5n/IPMIgtP0Xyk21H3YQ6fTuDG4QY+07UAU++2X6lS7oQ0O00OFh2ddx1+Q6RYrHpdJoOiaN5UqQffPXP3/oUGIel+MAAtord8Iod7InYrCXSq4msAV/XeeuamCnkFXg0dyGYC783hENR9ql5ESV8sSAfF0vl43E6vsWT8qGq3V1KeflIipbej38PlG5qsAB+CdG+FQQpB3h5/PpUi3X01FBk= Alfredo.Mendoza@CTCVM4929"
        ]
      }
    ]
  },
  "storage": {
    "files": [
      {
        "overwrite": true,
```
