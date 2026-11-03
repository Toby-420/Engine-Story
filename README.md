# Engine Story
A declarative, engine-agnostic, progressively compliant domain-specific language for visual novels.

## 1. Overview
Engine Story is a declarative scripting language designed for writing visual novels. It describes what the outcome of each action must be rather than steps to be taken.

Engines may interpret and take actions in the manner they choose, providing the outcome conforms with this specification.

The language is designed to be progressively compliant, meaning engines can support subsets of functionality to suit needs (such as platform or size requirements) without causing compatiblity issues.

### 1.1. File Formats
Engine Story can have the following file extensions (case insensitive):
- .EST (recommended, Engine STory)
- .STY (legacy, engine STorY)

Of course, the file extension has no bearing of the actual contents of the file, but since it is used on non-unix systems, file extensions are important.

Due to the format being used on legacy systems such as DOS, the maximum length of a new story file is 8 characters. This allows for maximum portability of stories. Since, if using a sensible yet terse naming convention, a scenario name is unlikely to be more than 8 characters, this is unlikely to cause issues.
It is important to remember than comments can be used to explain unclear code.

The line length required by Engine Story is exactly 256 characters.
Every implementation must safely take in each line so as to not crash if a line goes over the maximum length.

The character encoding of the file must be UTF-8 as it is compatible with most systems.
Newline must be CRLF as it is most portable.

## 2. Conformance Levels
Engines are classified according to their supported commands

### 2.1. Generic Classes
Basic: ` ! $

Type-1: Basic + @ : ?

Type-2: Type-1 + $$ _ / ;NUM ; #

Type-3: Type-2 + * ** *@

### 2.2. Additions
A: ~ ~!

V: %

R: implements all of section 6.

Additions are independent of generic classes.

Conformance is therefore displayed as such:
- Basic_A (conforms to Basic and implements audio handling)
- Type-3_AV (conforms to Type-3 and implements both audio and image handling)
- Type-2_ARV (conforms to Type-2, implements both audio and image handling, and implements all of section 5.)

## 3. Behaviour
### 3.1. Sequential Execution
Engine Story is sequentially executed. The order the commands appear is the order they are executed.
Execution begins at the start of the file and proceeds until termination or a branch is encountered.

### 3.2. Branching
When branching is encountered, execution just continues from the specified point.
Every other state (images, audio, etc) are all left unmodified.

### 3.3. State-based Execution
Engine Story is a state-based language. Commands will only affect the state is it explicitly intended to control.
Commands must never implicitly modify any unrelated aspect.

For example, a dialogue command will never modify the currently playing audio.

The initial state of every command is blank.
This means:
- No default image
- No default speaker
- No default dialogue

### 3.4. Persistance
State changes persist until another command explicitly affecting the same state is encountered, or the engine terminates, or another relevant operation.

For example, music will not stop playing unless a music stop command is met, or the music reaches the end of its playtime and does not loop.

### 3.5. Command Recognition
Any dialogue is processed through the $ series of commands. This means there is no implicit processing of dialogue.
Content not starting with a recognised control character is treated as an unsupported command.

### 3.6. Unsupported Commands
If an unsupported command is encountered, the engine shall skip the entire command, optionally displaying a warning explaining what was encountered.
This allows story files written for higher compliance classes to be able to be run (albeit with fewer frills) on lower compliance engines.

### 3.7. Incorrect Labels
If a label is specified that doesn't exist, the engine shall terminate and optionally display an error specifying what went wrong.

## 4. Command Definitions
Commands take up 1 line and must begin with a recognised control character.
Commands may then be followed immediately by a modifier character.
Then, if an argument is required, it is provided.

### 4.1. Command Families
` Comment

/ Speaker

$ Dialogue

; Typewriter Mode

@ Label Definitions

: Jump Handling (aka. goto)

? User Choice Branch

% Image Display

~ Music Handling

\* File Handling

! Termination

### 4.2. Modifiers
#### 4.2.1. Dialogue
$ Don't wait for keypress
# Newline (within dialogue line)
#### 4.2.2. Music handling
! Stop currently playing music
#### 4.2.3. File Handling
\* Return to previous file
@ Story Loading

### 4.3. Arguments
#### 4.3.1. Speaker
Defines speaker name
#### 4.3.2. Dialogue
Defines dialogue to be displayed
#### 4.3.3. Typewriter Mode
Defines typewriter delay (milliseconds)
#### 4.3.4. Label Definitions
Defines label name
#### 4.3.5. User Choice Branch
Defines displayed choices and corresponding labels
#### 4.3.6. Image Display
Defines image to be displayed (no file extension)
#### 4.3.7. Music Handling
Defines music to be played (no file extension)
#### 4.3.8. File Handling
Defines file to be opened (no file extension)

### 4.4. Further Information
#### 4.4.1. Dialogue
$$ will cause the dialogue to not wait for a keypress.
#### 4.4.2. Typewriter Mode
; on its own acts as ;0
This is shorthand for turning off the delay.
#### 4.4.3. User Choice Branch
The full syntax for this is as follows:

?Choice 1|Label1&Choice 2|Label2

This will show the user "Choice 1" and "Choice 2" and will go to Label1 if "Choice 1" is chosen and Label2 if "Choice 2" is chosen.

This means choices and labels cannot contain | or &.
Labels are case-sensitive, but labels should be defined in CAPITALS to make them clear as labels.

There can theoretically be as many choices added in as the author wants, but most branches are unlikely to use more than 3 or 4.

#### 4.4.3. File Handling
To load a specific file at a specific line number, use the following syntax:

*@FILE:LINENUM

To return to the previous file, use:

**

This does not keep a stack of files visited, only the previous one. Therefore, if it is called consecutively, it shall cause a loop of sorts.

## 5. Lexical and Syntactical Rules
### Whitespace
The only command supporting whitespace between the control character and the argument is dialogue as it provides clarity.
### Case Sensitivity
The entire language is internally case sensitive. However, some platforms are not case sensitive. It is best practice to treat everything as case sensitive as it offers most portability.

## 6. Recommendations
Nothing in this section is required for compliance, but it earns the relevant 'badge' as mentioned in 2.2.
If the engine does not implement a command with recommended aspects, it does not need to implement anything for that section. For example, an engine with Basic_R compliance only needs to worry about story file load order.

### 6.1. Error and Warning Messages
The engine shall output a warning or error message to the terminal (or equivalent for the platform) depending on the encountered issue. Examples are 3.6. and 3.7.
### 6.2. File Priority
Since no file extensions are given for file loading, priorities can be made for loading.
#### 6.2.1. Story Files
These should be loaded in the order given in 1.1.
#### 6.2.2. Image Files
##### 6.2.2.1. General Implementations
On modern, standard systems, the following advice should be followed:

Only the formats specified here need to be handled.
These should be loaded in the following order:

1 - JPEG

2 - PNG

To reach recommended status for music files, an engine must not load proprietary formats as these often require licenses which can stop some users from using them. Since there are so many high quality open formats, there is no reason to use a proprietary format.

##### 6.2.2.2. Specific Implementations
For legacy systems or systems where standard files (such as JPEG) will not work, it is up to the implementation author to decide whether to use a standard for that system, or to create a new format that fills their requirements.
In this case, the implementation author should provide or link to free and open source tooling to convert between ordinary files and these specific files along with an explanation stating how the new files work.

#### 6.2.3. Music Files
##### 6.2.3.1. General Implementations
On modern, standard systems, the following advice should be followed:

Only the formats specified here need to be handled to gain Recommended status.

The order should then be:

1 - ogg

2 - mp3

3 - flac

4 - wav

To reach recommended status for music files, an engine must not load proprietary formats as these often require licenses which can stop some users from using them. Since there are so many high quality free formats, there is no reason to use a proprietary format.

##### 6.2.3.2. Specific Implementations
Read 6.2.2.2.
