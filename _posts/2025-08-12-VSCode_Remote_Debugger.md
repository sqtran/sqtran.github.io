---
title: VS Code Remote Debugger
date: 2025-08-12
tags: ocp devspaces vscode debugger
---

## Configuring Remote Debugging in VSCode (in Dev Spaces)

This is a walkthrough of setting up the interactive debugger that connects to a Remote Program, such as a Web Container like Tomcat, JBoss EAP, IBM Websphere Liberty, or any other type of server that accepts remote connections.  As long as the server you're connecting to is listening, it will probably work.

Open the debugger menu on the left hand menu (it's the Play icon with a bug at the bottom corner)

![Open](https://lh3.googleusercontent.com/pw/AP1GczPz7tynSFHK9o_byNUUSGw-lgEreRRMvci7Tk7pqe8CGuc_w0YQuzNhnK7vW6JhYHyskY3Lp-UaznHuZzr9bCK9M0x_M4vJT70EkjEwWFAnKMzqWzUToUf4hirosEVPL2G_2qkymgOxcP5YcAQkm8oOUw=w431-h276-s-no-gm?authuser=0)

This Popup may appear if you're running in Java Light mode.  Pressing Yes enables more Java functionality and tooling, and will start compiling in the background.

![Confirm Java](https://lh3.googleusercontent.com/pw/AP1GczO4ExD79mkFTJeN85Ruq8p6XdxgVoi3l9U9jI_QTlkPYpnsB0g5XXMUeffSddv_aatXWTItRsFHb6EH9w9yXx0W_0mI9JKQq87T_LXFRf9pu-Dgp6fVBTfr8358u3vfZsADOKduf3pnoSx1dzk0gKLM3A=w529-h186-s-no-gm?authuser=0)

Expand the Current File dropdown to add a new Config.

![Add Config](https://lh3.googleusercontent.com/pw/AP1GczPigve4DfRgX_kfeM-VITWY2cGsJZ6RP00brUvriejQvMfl2R5lhNgsG5rQBDRba4yKrueMFGVrGGTj9EpdEKgqCuHtQmYT4nrqQemxDoBsGCkTsuTMqm8vQvGYqhBYLnp43Zb3wrn9OCjy5u7YcdBA8g=w628-h112-s-no-gm?authuser=0)

It creates a `launch.json` file if one doesn't already exists in your workspace.  Click anywhere in the file and a context menu will open.  Select the "Java: Attach to Remote Program" option.

![Edit Config](https://lh3.googleusercontent.com/pw/AP1GczPFepr0YnVuxjYYNbKspZHPwhsIN75dUlI0aaW-jIbxhJWXVmvGk10JC9NikrMUsdAK7sg7s0jmyW1ukqsNCfPd6cKdLm86LMRT2MxUpVx9aS_Svxeb-8PBLA7KUQrWHgZNe1ra16OPgdoUb4UN1iczgQ=w624-h279-s-no-gm?authuser=0)

Plug in the hostName and port into the configs like the following.

![Update](https://lh3.googleusercontent.com/pw/AP1GczNMxggS42dM6Uuhbpz1kmw3tOXduM9ijyDFWp56NksmCVc5vFsjV8eXtOTkp-GSk3vMuKaAP3aGGAjmZL1hYHdBY11G0WuswZCHOkyJ9IOzkxoVjSqaTH-LY9QKsq5A_Mn-LqOAMrGOixLDhU5Z4_VrDg=w566-h280-s-no-gm?authuser=0)

Start your Remote Server and and press the Play button in the debugger here to attach.

![Debug](https://lh3.googleusercontent.com/pw/AP1GczNyd21qdTnfED88iYVBrC4cOVQR9eft_MKgxHhlbDorO_M6dNJ52VWUPYaGHZW_mh6i3a1qsD3fRxLoSzjCQ4SRp7_eNYAkidWzIEf8u78Qz3betspQY3FKlfSqo2gsqgGy7W1Q6OySsO6wUycMuLUlcA=w703-h137-s-no-gm?authuser=0)

Now you can add breakpoints at critical areas, walkthrough/step into functions, check runtime variables, and all the other remote debugging features.  Best of all, this works in DevSpaces too, so you really can have a one-stop shop for it all.