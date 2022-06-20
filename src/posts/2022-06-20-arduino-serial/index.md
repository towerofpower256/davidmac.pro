---
title: "How I gave my IoT Arduino a command-line interface"
description: I wanted to be able to talk to an Arduino controller, make changes and make it do things without needing to reprogram the damn thing every time. I suceeded! Here's how I did it.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2022-06-20
tags:
- Electronics
- Coding
eleventyExcludeFromCollections: false
---

## The problem
It's easy to reflash and reprogram your controller when you're playing around with it and it's always connected to your computer with the Arduino IDE installed.

But what happens when you've installed the controller in something that's nowhere near your computer and you want to reconfigure or change things? Like an IoT controller for your garage door. 

It's tedious and annoying when you have to:
1. pull it out of whatever it's in
1. plug it into your main computer
1. reprogram it
1. reinstall it back into wherever you removed it from

[![Flash it a lot](flash-alot-meme.png)](flash-alot-meme.png)

## The serial command-line interface
A solution to this problem is to give the controller a command-line interface over the serial connection, and use it to update the controller's configuration.

This allows you to make changes to the controller and to make the controller do things without needing to reprogram it. It still needs a computer, but you won't need a programmer on it, any ol' computer with a USB port will do.

All of the source code for my attempts can be found on my GitHub:
https://github.com/towerofpower256/MyArduinoSerialInterface

### Version 1 - simple single character
A very simple CLI interface. If the character `b` is sent over the serial connection, it will toggle the LED.
The CLI is blocking, and nothing else is able to run on the controller except for the CLI.

```
> bbbbb
ON
OFF
ON
OFF
ON
```

<details>
  <summary>Click to see the source code</summary>

```cpp
// MySerialInterface v1
// Using single characters from a serial interface to
// turn the LED on and off.
// By David McDonald 17/05/2022

#define LEDPIN LED_BUILTIN

bool ledOn = false;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");
  pinMode(LEDPIN, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  while (Serial.available()) {
    char c = Serial.read();
    if (c == 'b') {
      toggleLed();
    }
  }
}

void toggleLed() {
  ledOn = !ledOn;
  digitalWrite(LEDPIN, ledOn);
  if (ledOn) Serial.println("ON"); else Serial.println("OFF");
}
```
</details>

### Version 2 - single character switch
Another simple CLI similar to version one, but this time the character `0` will turn off the LED, and the character `1` will turn it on.
The CLI is still blocking, and doesn't allow anything else to run on the controller except for the CLI.

```
> 1001110
ON
OFF
OFF
ON
ON
ON
OFF
> 101
ON
OFF
ON
```

<details>
<summary>Click to see the source code</summary>

```cpp
// MySerialInterface v2
// Turn the LED on and off by sending 0 and 1 over serial
// By David McDonald 17/05/2022

#define LEDPIN LED_BUILTIN

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");
  pinMode(LEDPIN, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  while (Serial.available()) {
    char c = Serial.read();
    if (c == '0') {
      Serial.println("OFF");
      digitalWrite(LEDPIN, LOW);
    }
    if (c == '1') {
      Serial.println("ON");
      digitalWrite(LEDPIN, HIGH);
    }
  }
}
```

</details>

### Version 3 - single character switch, non-blocking
Same as before, sending 0 and 1 characters over serial, but is now non-blocking and allows the controller to work on other activities alongside the serial interface.
To demonstrate, it will output `millis` every 1000 milliseconds alongside the serial interface.

```
1000
2000
> 00101
OFF
OFF
ON
OFF
ON
3000
4000
5000
> 01
OFF
ON
6000
7000
```

<details>
<summary>Click to see the source code</summary>

```cpp
// MySerialInterface v3
// Turn the LED on and off by sending 0 and 1 over serial.
// Now non-blocking, doesn't jam up the controller waiting for serial.
// By David McDonald 17/05/2022

#define LEDPIN LED_BUILTIN
#define MILLIS_INTERVAL 1000

unsigned long lastMillis = 0;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");
  pinMode(LEDPIN, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  doSerial();
  doMillis();
}

void doMillis() {
  if ((millis() - lastMillis) < MILLIS_INTERVAL) return;
  lastMillis = millis();
  Serial.println(millis());
}

void doSerial() {
  if (Serial.available()) {
    char c = Serial.read();
    if (c == '0') {
      Serial.println("OFF");
      digitalWrite(LEDPIN, LOW);
    }
    if (c == '1') {
      Serial.println("ON");
      digitalWrite(LEDPIN, HIGH);
    }
  }
}
```
</details>

### Version 4 - word commands, non-blocking
Now uses commands instead of just single characters. Each command is on a single line. The commands are also not case-sensitive, so you can go crazy on the SHIFT key and the commands will still work

* OK = prints the message `I am OK`.
* SET = set the value of the variable to something (e.g. `set abc123`).
* GET = prints the value of the variable e.g. `That value: abc123`.
* Unknown commands get the mesage `Unknown command`.

```
1000
2000
> Ok
I am OK
3000
4000
> set Test Value
5000
> GeT
That value: Test Value
6000
> BadCommand
Unknown command
7000
```

<details>
<summary>Click to see the source code</summary>

```cpp
// MySerialInterface v4
// Now uses commands instead of just a single character.
// GET and SET a value into a variable.
// By David McDonald 17/05/2022

#define LEDPIN LED_BUILTIN
#define MILLIS_INTERVAL 1000

#define CLI_BUFFER_SIZE 32 // Max length of the serial command buffer
#define CLI_MODE_WAITING 0
#define CLI_MODE_READING 1

unsigned long lastMillis = 0;

char cliBuffer[CLI_BUFFER_SIZE];
byte cliMode;
byte cliIdx;

char thatValue[CLI_BUFFER_SIZE];

void doMillis() {
  if ((millis() - lastMillis) < MILLIS_INTERVAL) return;
  lastMillis = millis();
  Serial.println(millis());
}

void cliReset() {
  cliMode = CLI_MODE_WAITING;
  cliIdx = 0;
  memset(cliBuffer, '\0', CLI_BUFFER_SIZE);
}

void doCli() {
  while (Serial.available()) {
    char c = Serial.read();

    // CLI is waiting for an actual character, not a space, newline, or carriage-return
    // On the ASCII chart, printable characters start at number 32
    // https://www.ascii-code.com/
    if (cliMode == CLI_MODE_WAITING) {
      if (c >= 32 && c != ' ') {
        cliMode = CLI_MODE_READING; // Start reading
      }
    }

    // CLI is reading
    if (cliMode == CLI_MODE_READING) {
      if (c == '\n' || c == '\r') {
        // Is an end of line. Execute the line.
        cliExec();
        return;
      }

      // Add the character to the buffer
      if (cliIdx < CLI_BUFFER_SIZE-1) {
        cliBuffer[cliIdx] = c;
        cliIdx++;

        // If the buffer is full, execute now
        if (cliIdx >= CLI_BUFFER_SIZE-1) {
          cliExec();
          return;
        }
      }
    }
  }
}

// Execute the CLI command
void cliExec() {
  Serial.print("EXEC> "); Serial.println(cliBuffer);

  if (cliIdx == 0) return; // Line buffer is empty, nothing to do here

  char delim[] = " "; // Delimiters, should just be space characters
  char eolDelim[] = "\r\n";
  char *part;
  char *saveptr;
  
  // Get the first part
  part = strtok_r(cliBuffer, delim, &saveptr);
  strupr(part); // Make uppercase, so command is not case sensitive. "set" and "SeT" is the same as "SET".

  // Take action, depending on what the command was
  // strcmp compares 2 char arrays. If they are the same, the result will be zero.
  if (!strcmp(part,"OK")) {
    Serial.println("I am OK");
    
  } else if (!strcmp(part,"SET")) {
    // Get the next part.
    // Use NULL to resume from where you started in the line buffer string
    // This time, don't stop until the end of the line using eolDelim
    part = strtok_r(NULL, eolDelim, &saveptr);

    // Save that value
    setThatValue(part); 
    
  } else if (!strcmp(part,"GET")) {
    Serial.print("That value: '"); Serial.print(getThatValue()); Serial.println('\'');
    
  } else {
    Serial.println("Unknown command");
  }

  // Cleanup to get ready for the next line
  cliReset();
}

void setThatValue(char* newValue) {
  strcpy(thatValue, newValue);
}

char* getThatValue() {
  return thatValue;
}


void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");
  cliReset();
}

void loop() {
  // put your main code here, to run repeatedly:
  doCli();
  doMillis();
}
```
</details>

### Version 5 - word commands and paramters
A relatively complete serial interface, passing CLI details between functions, each command is its own function instead of everything bunched into a single function.

I got rid of the `millis` being outputted to the console, I've proven that that part works just fine.

Commands control the onboard LED.
* OK = prints the message `I am OK`.
* SET = Sets the LED. `SET ON` turns it on, `SET OFF` turns it off.

```
> OK
I am OK
> BadCommand
Unknown command
> set on
LED is ON
> sEt oN
LED is ON
> set off
LED is OFF
> set asdasdads

> set ddddddd

```

<details>
<summary>Click to see the source code</summary>

```cpp
// MySerialInterface v5
// A relatively complete serial interface, using functions to the commands, passing CLI details between scopes.
// GET and SET a value into a variable.
// By David McDonald 20/05/2022

#define LEDPIN LED_BUILTIN

#define CLI_BUFFER_SIZE 32 // Max length of the serial command buffer
#define CLI_MODE_WAITING 0
#define CLI_MODE_READING 1

char cliBuffer[CLI_BUFFER_SIZE];
byte cliMode;
byte cliIdx;
char *cliPartPtr;
char *cliPart;
const char cliDelimSpace[] = " ";
const char cliDelimEol[] = "\r\n";

bool ledValue = false;
const char ledValOn[] = "ON";
const char ledValOff[] = "OFF";

void cliReset() {
  cliMode = CLI_MODE_WAITING;
  cliIdx = 0;
  memset(cliBuffer, '\0', CLI_BUFFER_SIZE);
}

void doCli() {
  while (Serial.available()) {
    char c = Serial.read();

    // CLI is waiting for an actual character, not a space, newline, or carriage-return
    // On the ASCII chart, printable characters start at number 32
    // https://www.ascii-code.com/
    if (cliMode == CLI_MODE_WAITING) {
      if (c >= 32 && c != ' ') {
        cliMode = CLI_MODE_READING; // Start reading
      }
    }

    // CLI is reading
    if (cliMode == CLI_MODE_READING) {
      if (c == '\n' || c == '\r') {
        // Is an end of line. Execute the line.
        cliExec();
        return;
      }

      // Add the character to the buffer
      if (cliIdx < CLI_BUFFER_SIZE-1) {
        cliBuffer[cliIdx] = c;
        cliIdx++;

        // If the buffer is full, execute now
        if (cliIdx >= CLI_BUFFER_SIZE-1) {
          cliExec();
          return;
        }
      }
    }
  }
}

// Execute the CLI command
void cliExec() {
  Serial.print("EXEC> "); Serial.println(cliBuffer);

  if (cliIdx == 0) return; // Line buffer is empty, nothing to do here
  
  // Get the first part
  cliPart = strtok_r(cliBuffer, cliDelimSpace, &cliPartPtr);
  strupr(cliPart); // Make uppercase, so command is not case sensitive. "set" and "SeT" is the same as "SET".

  // Take action, depending on what the command was
  // strcmp compares 2 char arrays. If they are the same, the result will be zero.
  if (!strcmp(cliPart,"OK")) {
    cliOK();
    
  } else if (!strcmp(cliPart,"SET")) {
    cliSetLed();
  } else {
    Serial.print("Unknown command "); Serial.println(cliPart);
  }

  // Cleanup to get ready for the next line
  cliReset();
}

void cliOK() {
  Serial.println("I am OK");
}

void cliSetLed() {
  const char cliDelimSpaceEol[] = " \r\n";
  cliPart = strtok_r(NULL, cliDelimSpaceEol, &cliPartPtr);
  strupr(cliPart);
  
  if (!strcmp(cliPart, ledValOn)) {
    setLed(true);
  } else if (!strcmp(cliPart, ledValOff)) {
    setLed(false);
  }
}

void setLed(bool a) {
  ledValue = a;
  digitalWrite(LEDPIN, a);
  Serial.print("LED is ");
  if (ledValue) {
    Serial.println(ledValOn);
  } else {
    Serial.println(ledValOff);
  }
}

// =====================================
// MAIN
// =====================================

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");
  cliReset();
}

void loop() {
  // put your main code here, to run repeatedly:
  doCli();
}
```
</details>

### Version 6 - a command line interface class
The big one.

The CLI interface is built out into a object oriented library, and broken out into separate files instead of one massive file.

I learned a bit about how C++ memory management in this one, and how complicated it can get when passing objects through functions. I had problems where it's garbage-collected my object and used the memory for something else right out from under me, making weird things happen. But I sorted it out!

```
START
ADD CMD 0 OK
ADD CMD 1 SET
ADD CMD 2 MILLIS
> OK
I am OK
> MilLis
3165
> set asdadsasd
UNK LED CMD
> set on
LED ON
> set off
LED is OFF
> set off
LED is OFF
> asdasdasd
UNK CMD
> help
--- HELP ---
0 OK
1 SET
2 MILLIS
```

Commands are defined on startup. Each command is a `MyCliCmd` object, and uses callback functions to handle the incoming command. The command buffer is a `MyCliBuffer` object with helper functions for reading the buffer.

```cpp
void cliOK(char *cmd, char* buffer) {
  // ...
}

void cliMillis(char *cmd, char* buffer) {
  // ...
}

void cliSetLed(char *cmd, char* buffer) {
  // ...
}

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");

  // Setup the CLI
  cli = MyCli();

  cli.addCmd("ok", &cliOK);
  cli.addCmd("set", &cliSetLed);
  cli.addCmd("millis", &cliMillis);
}

void loop() {
  // put your main code here, to run repeatedly:
  cli.think();
}
```

Here's an example CLI command for setting the onboard LED.

```cpp
void cliSetLed(char *cmd, char* buffer) {
  char* cliPart = strtok(buffer, CLI_DELIM_SPACE_EOL);
  strupr(cliPart); // Make uppercase, don't care about case sensitivity
  
  if (!strcmp(cliPart, ledValOn)) {
    setLed(true);
  } else if (!strcmp(cliPart, ledValOff)) {
    setLed(false);
  } else {
    Serial.println("UNK LED CMD");
  }
}
```

<details>
<summary>Click to see the source code</summary>

MyCli.h
```cpp
#define CLI_CMD_NUM_MAX 6 // Max size of command list
#define CLI_BUFFER_SIZE 32 // Max length of the serial command buffer

#define CLI_MODE_WAITING_START 0
#define CLI_MODE_READING 1
#define CLI_MODE_WAITING_EOL 2


class MyCli {
  public:
    MyCli();
    void reset();
    void think();
    void execLine();
    void printHelp();
    void addCmd(char *cmd, void (*callback)(char*, char*));
    byte cliMode;
  private:
    char buffer[CLI_BUFFER_SIZE];
    byte bufferIdx;
    void bufferReset();
    
    //MyCliCmd* cmdList[CLI_CMD_NUM_MAX];
    byte cmdListIdx;
    char* cmdListCmd[CLI_CMD_NUM_MAX];
    void (*cmdListCallback[CLI_CMD_NUM_MAX])(char*, char*);
};

MyCli::MyCli() {
  cliMode = CLI_MODE_WAITING_START;
  memset(buffer, 0, CLI_BUFFER_SIZE);
  
  cmdListIdx = 0;
  memset(cmdListCmd, 0, sizeof(cmdListCmd));
  memset(cmdListCallback, 0, sizeof(cmdListCallback));

  bufferReset();
}

void MyCli::printHelp() {
  Serial.println(F("--- HELP ---"));
  for (byte iCmd=0; iCmd < cmdListIdx; iCmd++) {
    Serial.println(this->cmdListCmd[iCmd]);
  }
}

// DOESN'T WORK
// Because this '_cmd' reference dies with this function call, the object is garbage collected,
// even through we're storing a reference to it.
// Then, when we run this to make another CMD object, the new one takes the same place in
// memory as the old one. The end result is a big list of the same objects, a list of CMD
// pointers, all pointing to the same place in memory.
// Yes, I could get around this by using 'malloc' to forcefully allocate the space in memory,
// but I don't want to open that pandora's box, and risk a memory leak.
// E.g.
// [0] 0x08CA MILLIS
// [1] 0x08CA MILLIS
// [2] 0x08CA MILLIS
/*
void MyCli::addCmd(char *cmd, void (*callback)(char*, MyCliBuffer&)) {
  MyCliCmd* _cmd = (MyCliCmd*)malloc(sizeof(MyCliCmd));
  _cmd = &MyCliCmd(cmd, callback);
  this->addCmd(_cmd);
}
*/


void MyCli::addCmd(char* cmd, void (*_callback)(char*, char*)) {
  // Don't add a CMD if the list is full
  if (cmdListIdx >= CLI_CMD_NUM_MAX) {
    Serial.println("ERR CMD LIST FULL");
    return;
  }

  // Make commands upper case, to ignore case sensitivity
  strupr(cmd);

  Serial.print("ADD CMD "); Serial.print(cmdListIdx); Serial.print(CHAR_SPACE); Serial.println(cmd);
  cmdListCmd[cmdListIdx] = cmd;
  cmdListCallback[cmdListIdx] = _callback;
  cmdListIdx++;
}

void MyCli::think() {
  while (Serial.available()) {
    char c = Serial.read();
    //Serial.print(bufferIdx); Serial.print(' '); Serial.println(c);

    // CLI is waiting for an actual character, not a space, newline, or carriage-return
    // On the ASCII chart, printable characters start at number 32
    // https://www.ascii-code.com/
    if (cliMode == CLI_MODE_WAITING_START) {
      if (c >= 32 && c != ' ') {
        cliMode = CLI_MODE_READING; // Start reading
        //Serial.println("SREAD");
      }
    }

    // CLI is waiting for the end of line. Previous line was probably bad.
    if (cliMode == CLI_MODE_WAITING_EOL) {
      if (c == '\n' || c == '\r') {
        cliMode = CLI_MODE_WAITING_START;
        return;
      }
    }

    // CLI is reading
    if (cliMode == CLI_MODE_READING) {
      if (c == '\n' || c == '\r') {
        // Is an end of line. Execute the line.
        //Serial.println("EOL");
        this->execLine();
        return;
      }

      // Add the character to the buffer
      if (bufferIdx < CLI_BUFFER_SIZE) {
        //Serial.println("ADD C");
        buffer[bufferIdx] = c;
        bufferIdx++;

        // If the buffer is full
        if (bufferIdx >= CLI_BUFFER_SIZE-1) {
          // Overflow
          cliMode = CLI_MODE_WAITING_EOL;
          return;
        }
      }
    }
  }
}

// Execute the CLI command
void MyCli::execLine() {
  Serial.print("EXEC> "); Serial.println(buffer);

  if (bufferIdx == 0) {
    // Line buffer is empty, nothing to do here
    Serial.println("EMPTY");
    return; 
  }

  // Get the first part, ready to the first space character
  char delim[] = " \r\n";
  char* cliPartPtr;
  char* cliPart = strtok_r(buffer, delim, &cliPartPtr);
  strupr(cliPart); // Make uppercase, so command is not case sensitive. "set" and "SeT" is the same as "SET".
  Serial.print("CMD> "); Serial.println(cliPart);

  // Loop through commands, looking for one with a matching command.
  // Once found, run it
  bool ranCmd = false;
  if (!strcmp("HELP", cliPart)) {
    this->printHelp();
    ranCmd = true;
  }

  if (!ranCmd) {
    for (byte iCmd=0; iCmd < cmdListIdx; iCmd++) {
      //Serial.print("T ");Serial.print(iCmd); Serial.println(cmdListCmd[iCmd]);
      if (!strcmp(cmdListCmd[iCmd], cliPart)) {
        // Found a match. Run it.
        //Serial.print("Running cmd: "); Serial.println(cliPart);
        cmdListCallback[iCmd](cliPart, cliPartPtr);
        ranCmd = true;
      }
    }
  }
  
  if (!ranCmd) {
    Serial.println("UNK CMD");
  }
  
  // Cleanup to get ready for the next line
  this->reset();
}

void MyCli::reset() {
  cliMode = CLI_MODE_WAITING_START;
  bufferReset();
}

void MyCli::bufferReset() {
  bufferIdx = 0;
  memset(buffer, 0, CLI_BUFFER_SIZE);
}
```

MySerialInterfaceV6.ino
```cpp
// MySerialInterface v6
// A command line library as a self-contained extensible object.
// By David McDonald 27/05/2022

#define CHAR_SPACE " "

#include "MyCli.h"

char CLI_DELIM_SPACE[] = CHAR_SPACE;
char CLI_DELIM_EOL[] = "\r\n";
char CLI_DELIM_SPACE_EOL[] = " \r\n";

// ======================================================
// Main program
// ======================================================

#define LEDPIN LED_BUILTIN

bool ledValue = false;
const char ledValOn[] = "ON";
const char ledValOff[] = "OFF";


void cliOK(char *cmd, char* buffer) {
  Serial.println("I am OK");
}

void cliSetLed(char *cmd, char* buffer) {
  char* cliPart = strtok(buffer, CLI_DELIM_SPACE_EOL);
  strupr(cliPart); // Make uppercase, don't care about case sensitivity
  Serial.print("LED part "); Serial.println(cliPart);
  
  if (!strcmp(cliPart, ledValOn)) {
    setLed(true);
  } else if (!strcmp(cliPart, ledValOff)) {
    setLed(false);
  } else {
    Serial.println("UNK LED CMD");
  }
}

void cliMillis(char *cmd, char* buffer) {
  Serial.println(millis());
}

void setLed(bool a) {
  ledValue = a;
  digitalWrite(LEDPIN, a);
  Serial.print("LED ");
  if (ledValue) {
    Serial.println(ledValOn);
  } else {
    Serial.println(ledValOff);
  }
}

// =====================================
// MAIN
// =====================================

MyCli cli;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");

  // Setup the CLI
  cli = MyCli();

  cli.addCmd("ok", &cliOK);
  cli.addCmd("set", &cliSetLed);
  cli.addCmd("millis", &cliMillis);
}

void loop() {
  // put your main code here, to run repeatedly:
  cli.think();
}
```
</details>

### Version 7 - a command line interface class and settings
Similar to the previous version with a library function to keep things consistent, this version includes a `MySettings` object full of configurations, and there are CLI commands to get and set the configuration.

This is very close to my goal of a configuration that can be updated on-the-fly, and without needing to re-flash the controller any time the configuration has changed.

I'm really proud of this one.

```
START
ADD CMD 0 OK
ADD CMD 1 LED
ADD CMD 2 MILLIS
ADD CMD 3 SET
ADD CMD 4 GET
EXEC> ok
I am OK
EXEC> led on
LED ON
EXEC> led off
LED OFF
EXEC> get
--- CONFIG ---
WTIMEOUT	60
WSSID	
WPWD	
EXEC> set wtimeout 25
CONF: WTIMEOUT
OK
EXEC> set wssid MyHomeNetwork
CONF: WSSID
OK
EXEC> set wpwd co07Pas@worD
CONF: WPWD
OK
EXEC> get
--- CONFIG ---
WTIMEOUT	25
WSSID	MyHomeNetwork
WPWD	co07Pas@worD
```

<details>
<summary>Click to see the source code</summary>

MyCli.h
(mostly unchanged from v6)

MySerialInterfaceV7.ino
```cpp
// MySerialInterface v7
// A command line library as a self-contained extensible object.
// But this time, with commands for setting and getting configuration.
// By David McDonald 20/06/2022

#define CHAR_SPACE " "

#include "MyCli.h"

char CLI_DELIM_SPACE[] = CHAR_SPACE;
char CLI_DELIM_EOL[] = "\r\n";
char CLI_DELIM_SPACE_EOL[] = " \r\n";
char MSG_OK[] = "OK";
char MSG_OVERSIZE_PARM[] = "Parm too long";
char MSG_INVALID_NUMBER[] = "Invalid number";

// ======================================================
// Global and settings
// ======================================================

#define CONFIG_STRING_SIZE 31

char CFG_WTIMEOUT[] = "WTIMEOUT";
char CFG_WSSID[] = "WSSID";
char CFG_WPWD[] = "WPWD";

typedef struct {
  int wifiTimeout;
  char wifiSSID[CONFIG_STRING_SIZE];
  char wifiPassword[CONFIG_STRING_SIZE];
} MySettings;

MySettings mySettings;

void setupSettings() {
  // Reset the settings back to default
  memset(&mySettings, 0, sizeof(MySettings));
  mySettings.wifiTimeout = 60;
}

void cliSetConfig(char* cmd, char* buffer) {
  char* cliPart = strtok(buffer, CLI_DELIM_SPACE);
  strupr(cliPart);
  Serial.print("CONF: "); Serial.println(cliPart);

  if (!strcmp(cliPart, CFG_WSSID)) {
    char strbuffer[CONFIG_STRING_SIZE];
    memset(strbuffer, 0, CONFIG_STRING_SIZE);
    cliPart = strtok(NULL, CLI_DELIM_SPACE_EOL);
    if (strlen(cliPart) < CONFIG_STRING_SIZE) {
      strcpy(mySettings.wifiSSID, cliPart);
      Serial.println(MSG_OK);
    } else {
      Serial.println(MSG_OVERSIZE_PARM);
    }

    return;
  } else if (!strcmp(cliPart, CFG_WPWD)) {
    char strbuffer[CONFIG_STRING_SIZE];
    memset(strbuffer, 0, CONFIG_STRING_SIZE);
    cliPart = strtok(NULL, CLI_DELIM_SPACE_EOL);
    if (strlen(cliPart) < CONFIG_STRING_SIZE) {
      strcpy(mySettings.wifiPassword, cliPart);
      Serial.println(MSG_OK);
    } else {
      Serial.println(MSG_OVERSIZE_PARM);
    }

    return;
  } else if (!strcmp(cliPart, CFG_WTIMEOUT)) {
    cliPart = strtok(NULL, CLI_DELIM_SPACE_EOL);
    if (cliPart == '0') { 
      mySettings.wifiTimeout = 0;
      Serial.println(MSG_OK);
      return;
    }
    
    int intBuffer = atoi(cliPart);
    if (intBuffer == 0) {
      Serial.println(MSG_INVALID_NUMBER);
    } else {
      mySettings.wifiTimeout = intBuffer;
      Serial.println(MSG_OK);
    }
    return;
  } else {
    Serial.println(F("Unknown config"));
    return;
  }
}

void cliGetConfig(char* cmd, char* buffer) {
  Serial.println(F("--- CONFIG ---"));
  Serial.print(CFG_WTIMEOUT); Serial.print('\t'); Serial.println(mySettings.wifiTimeout);
  Serial.print(CFG_WSSID); Serial.print('\t'); Serial.println(mySettings.wifiSSID);
  Serial.print(CFG_WPWD); Serial.print('\t'); Serial.println(mySettings.wifiPassword);
}

// ======================================================
// Main program
// ======================================================

#define LEDPIN LED_BUILTIN

bool ledValue = false;
const char ledValOn[] = "ON";
const char ledValOff[] = "OFF";


void cliOK(char *cmd, char* buffer) {
  Serial.println("I am OK");
}

void cliSetLed(char *cmd, char* buffer) {
  char* cliPart = strtok(buffer, CLI_DELIM_SPACE_EOL);
  strupr(cliPart); // Make uppercase, don't care about case sensitivity
  //Serial.print("LED part "); Serial.println(cliPart);
  
  if (!strcmp(cliPart, ledValOn)) {
    setLed(true);
  } else if (!strcmp(cliPart, ledValOff)) {
    setLed(false);
  } else {
    Serial.println("UNK LED CMD");
  }
}

void cliMillis(char *cmd, char* buffer) {
  Serial.println(millis());
}

void setLed(bool a) {
  ledValue = a;
  digitalWrite(LEDPIN, a);
  Serial.print("LED ");
  if (ledValue) {
    Serial.println(ledValOn);
  } else {
    Serial.println(ledValOff);
  }
}

// =====================================
// MAIN
// =====================================

MyCli cli;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("START");

  // Setup settings
  setupSettings();

  // Setup the CLI
  cli = MyCli();

  cli.addCmd("ok", &cliOK);
  cli.addCmd("led", &cliSetLed);
  cli.addCmd("millis", &cliMillis);
  cli.addCmd("set", &cliSetConfig);
  cli.addCmd("get", &cliGetConfig);
}

void loop() {
  // put your main code here, to run repeatedly:
  cli.think();
}
```
</details>