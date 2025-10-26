# TEC-1 EtchABot Controller - Complete Guide

<img width="990" height="660" alt="image" src="https://github.com/user-attachments/assets/d3c78591-010f-41f0-9686-7f8f1e092275" />

![Etchasketch](https://github.com/user-attachments/assets/0f841757-604a-4950-91dd-cfc199b2cceb)



## Overview

This document provides a comprehensive guide for creating an Etch-A-Sketch-like drawing device using the TEC-1 computer with stepper motors. It includes Arduino-style pseudo code examples, Z80 assembly implementation, and MINT programming language solutions.

---

## Table of Contents

1. [Hardware Overview](#hardware-overview)
2. [Arduino-Style Implementation](#arduino-style-implementation)
3. [Z80 Assembly Implementation](#z80-assembly-implementation)
4. [MINT Programming Guide](#mint-programming-guide)
5. [Stepper Motor Control Examples](#stepper-motor-control-examples)
6. [G-Code Implementation](#g-code-implementation)

---

## Hardware Overview

The EtchABot system consists of:
- **TEC-1 Computer**: Main controller
- **Two Stepper Motors**: X and Y axis movement
- **Four End-Stop Switches**: X-min, X-max, Y-min, Y-max
- **Optional Pen Mechanism**: For actual drawing
- **Driver Circuits**: For stepper motor control

### Port Assignments (TEC-1)
- **Ports 3-6**: X-axis stepper motor (phases A, B, C, D)
- **Ports 7-10**: Y-axis stepper motor (phases A, B, C, D)
- **Ports 8-11**: End-stop switches (optional)

---

## Arduino-Style Implementation

### Basic Structure

```cpp
// Global Definitions
#define X_STEP_PIN 2
#define X_DIR_PIN  3
#define Y_STEP_PIN 5
#define Y_DIR_PIN  6

// End stop pins
#define X_MIN_PIN  8
#define X_MAX_PIN  9
#define Y_MIN_PIN  10
#define Y_MAX_PIN  11

// Motor parameters
#define STEPS_PER_MM 10
#define STEP_PULSE_US 5
#define STEP_DELAY_US 300

// Position tracking (16-bit signed)
int16_t currentX = 0;
int16_t currentY = 0;
int16_t xTravelMax = 0;
int16_t yTravelMax = 0;

void setup() {
    // Configure pins
    pinMode(X_STEP_PIN, OUTPUT);
    pinMode(X_DIR_PIN, OUTPUT);
    pinMode(Y_STEP_PIN, OUTPUT);
    pinMode(Y_DIR_PIN, OUTPUT);
    
    // Configure end stops with pullups
    pinMode(X_MIN_PIN, INPUT_PULLUP);
    pinMode(X_MAX_PIN, INPUT_PULLUP);
    pinMode(Y_MIN_PIN, INPUT_PULLUP);
    pinMode(Y_MAX_PIN, INPUT_PULLUP);
    
    Serial.begin(115200);
    homeAllAxes();
}

void loop() {
    if (Serial.available()) {
        String command = Serial.readString();
        processGCode(command);
    }
}
```

### Movement Functions

```cpp
void moveTo(int16_t targetX, int16_t targetY) {
    int16_t dx = targetX - currentX;
    int16_t dy = targetY - currentY;
    
    int16_t stepsX = abs(dx);
    int16_t stepsY = abs(dy);
    
    // Set direction pins
    digitalWrite(X_DIR_PIN, dx >= 0 ? HIGH : LOW);
    digitalWrite(Y_DIR_PIN, dy >= 0 ? HIGH : LOW);
    
    // Simultaneous movement (Bresenham-like)
    int16_t maxSteps = max(stepsX, stepsY);
    
    for (int i = 0; i < maxSteps; i++) {
        if (i < stepsX && !checkEndStop(X_AXIS, dx)) {
            stepMotor(X_STEP_PIN);
            currentX += (dx >= 0) ? 1 : -1;
        }
        
        if (i < stepsY && !checkEndStop(Y_AXIS, dy)) {
            stepMotor(Y_STEP_PIN);
            currentY += (dy >= 0) ? 1 : -1;
        }
        
        delayMicroseconds(STEP_DELAY_US);
    }
}

void stepMotor(int stepPin) {
    digitalWrite(stepPin, HIGH);
    delayMicroseconds(STEP_PULSE_US);
    digitalWrite(stepPin, LOW);
    delayMicroseconds(STEP_PULSE_US);
}

bool checkEndStop(int axis, int direction) {
    if (axis == X_AXIS) {
        if (direction < 0 && digitalRead(X_MIN_PIN) == LOW) return true;
        if (direction > 0 && digitalRead(X_MAX_PIN) == LOW) return true;
    } else {
        if (direction < 0 && digitalRead(Y_MIN_PIN) == LOW) return true;
        if (direction > 0 && digitalRead(Y_MAX_PIN) == LOW) return true;
    }
    return false;
}
```

---

## Z80 Assembly Implementation

### Core Structure

```assembly
; Z80 Assembly for Etch A Sketch Controller
; I/O Ports
X_STEP  EQU $00    ; X stepper step pin
X_DIR   EQU $01    ; X stepper direction pin
Y_STEP  EQU $02    ; Y stepper step pin
Y_DIR   EQU $03    ; Y stepper direction pin

; Memory locations
CUR_X   EQU $8000  ; Current X position
CUR_Y   EQU $8001  ; Current Y position
TARG_X  EQU $8003  ; Target X position
TARG_Y  EQU $8004  ; Target Y position

START:
        LD A, 0
        LD (CUR_X), A
        LD (CUR_Y), A
        CALL HOME
        JP MAIN_LOOP

MOVE_TO:
        ; Calculate deltas and move with interpolation
        ; Implementation details in full code...
        RET

HOME:
        ; Home both axes to origin
        ; Implementation details in full code...
        RET
```

---

## MINT Programming Guide

### Introduction to MINT

MINT (Minimalist Interpreter) is a stack-based programming language designed for the TEC-1. It uses Reverse Polish Notation (RPN) and provides direct hardware control capabilities.

### Basic Syntax Rules

1. **Variables**: Single lowercase letters `a` to `z` (26 total)
2. **Functions**: Single uppercase letters `A` to `Z` (26 total)
3. **Numbers**: 16-bit signed integers (-32768 to 32767)
4. **Hexadecimal**: Prefixed with `#` (e.g., `#FF00`)
5. **Functions**: Defined with `:` and ended with `;`
6. **Comments**: Use `//` on separate lines only

### Essential MINT Concepts

#### Stack Operations
```mint
10 20 +        // Add: results in 30
10 "           // Duplicate: stack has 10 10
10 20 $        // Swap: results in 20 10
10 20 '        // Drop: results in 10
```

#### Variables and Assignment
```mint
10 x!          // Assign 10 to variable x
x 5 + y!       // Add 5 to x, store in y
x .            // Print value of x
```

#### Arrays
```mint
[1 2 3 4 5] a!     // Create array, store pointer in a
a 2?               // Get element at index 2 (value 3)
10 a 2?!           // Set element at index 2 to 10
a /S               // Get array size
```

#### Loops and Conditionals
```mint
10( x . )          // Loop 10 times, print x each time
x 5 > ( `big` )    // If x > 5, print "big"
x 5 > ( `big` ) /E ( `small` )  // If-else statement
```

---

## Stepper Motor Control Examples

### Basic Motor Control Functions

```mint
// Initialize hardware ports
:A 3 p! 4 q! 5 r! 6 s! 7 t! 8 u! 9 v! 10 w! ;

// Initialize positions
:B 0 a! 0 b! ;

// Print current position
:C `X:` a . ` Y:` b . /N ;

// Delay functions
:D 10 x! ;
:E 50 x! ;

// X-axis stepper phases (4-phase sequence)
:F 1 p /O 0 q /O 0 r /O 0 s /O D ;  // Phase 1
:G 0 p /O 1 q /O 0 r /O 0 s /O D ;  // Phase 2
:H 0 p /O 0 q /O 1 r /O 0 s /O D ;  // Phase 3
:I 0 p /O 0 q /O 0 r /O 1 s /O D ;  // Phase 4

// Y-axis stepper phases
:J 1 t /O 0 u /O 0 v /O 0 w /O D ;  // Phase 1
:K 0 t /O 1 u /O 0 v /O 0 w /O D ;  // Phase 2
:L 0 t /O 0 u /O 1 v /O 0 w /O D ;  // Phase 3
:M 0 t /O 0 u /O 0 v /O 1 w /O D ;  // Phase 4

// Step X motor forward n times
:N n! n( F G H I ) ;

// Step X motor backward n times
:O n! n( I H G F ) ;

// Step Y motor forward n times
:P n! n( J K L M ) ;

// Step Y motor backward n times
:Q n! n( M L K J ) ;
```

### Position Control Functions

```mint
// Move to absolute position
:R 
x! y!                    // Get target coordinates
x a - dx!                // Calculate X delta
y b - dy!                // Calculate Y delta

// Handle X movement
dx 0 > ( dx N a dx + a! )     // Move X positive
dx 0 < ( 0 dx - O a dx + a! ) // Move X negative

// Handle Y movement  
dy 0 > ( dy P b dy + b! )     // Move Y positive
dy 0 < ( 0 dy - Q b dy + b! ) // Move Y negative

C                        // Print new position
;

// Move relative to current position
:S
dx! dy!                  // Get relative movement
a dx + x!                // Calculate new X
b dy + y!                // Calculate new Y
x y R                    // Move to new position
;

// Home both axes (move to 0,0)
:T
0 a - dx!                // Calculate distance to home X
0 b - dy!                // Calculate distance to home Y
0 0 R                    // Move to origin
;
```

### Advanced Movement with Interpolation

```mint
// Smooth diagonal movement using Bresenham-like algorithm
:U
x! y!                    // Get target coordinates
x a - dx!                // X delta
y b - dy!                // Y delta

// Determine which axis has more steps
dx dx 0 < ( 0 dx - dx! ) // Make dx positive
dy dy 0 < ( 0 dy - dy! ) // Make dy positive

dx dy > steps!           // Use larger delta as step count
dx dy <= ( dy steps! )

// Calculate step increments
dx 100 * steps / ix!     // X increment (scaled by 100)
dy 100 * steps / iy!     // Y increment (scaled by 100)

0 cx! 0 cy!              // Current accumulated positions

steps(                  // Loop for each step
  cx ix + cx!            // Accumulate X
  cy iy + cy!            // Accumulate Y
  
  // Check if time to step X
  cx 100 >= (
    x a > ( F G H I a 1 + a! )  // Step X forward
    x a < ( I H G F a 1 - a! )  // Step X backward
    cx 100 - cx!               // Reset accumulator
  )
  
  // Check if time to step Y
  cy 100 >= (
    y b > ( J K L M b 1 + b! )  // Step Y forward
    y b < ( M L K J b 1 - b! )  // Step Y backward
    cy 100 - cy!               // Reset accumulator
  )
  
  E                      // Delay between steps
)

C                        // Print final position
;
```

---

## G-Code Implementation

### Basic G-Code Parser

```mint
// G-code interpreter for drawing commands
:V                       // Main G-code interpreter
`G-code> `               // Prompt
/K cmd!                  // Read command character

// Handle G commands
cmd 71 = (               // ASCII 'G' = 71
  /K 48 - gnum!          // Get G-code number
  
  gnum 0 = gnum 1 = + (  // G0 or G1 (move)
    W                    // Call move parser
  )
  
  gnum 28 = (            // G28 (home)
    T                    // Call home function
  )
)

// Handle M commands  
cmd 77 = (               // ASCII 'M' = 77
  /K 48 - mnum!          // Get M-code number
  
  mnum 0 = (             // M0 (pen up)
    0 pen!
    `Pen UP` /N
  )
  
  mnum 1 = (             // M1 (pen down)
    1 pen!
    `Pen DOWN` /N
  )
)

V                        // Loop back for next command
;

// Parse X Y coordinates for movement
:W
`X` /K /K 48 - 10 * /K 48 - + x!  // Parse X coordinate
`Y` /K /K 48 - 10 * /K 48 - + y!  // Parse Y coordinate

pen 1 = (                // If pen is down
  x y U                  // Use smooth movement
) /E (                   // If pen is up
  x y R                  // Use rapid movement
)
;

// Initialize system
:X
A                        // Initialize ports
B                        // Initialize positions
`EtchABot Ready` /N
`Commands: G0/G1 XnYn, G28, M0, M1` /N
V                        // Start G-code interpreter
;
```

### Example G-Code Programs

#### Square Drawing Program
```mint
// Draw a 20x20 square
:Y
`Drawing Square...` /N
T                        // Home first
1 pen!                   // Pen down
20 0 U                   // Right 20 units
20 20 U                  // Up 20 units  
0 20 U                   // Left 20 units
0 0 U                    // Down to start
0 pen!                   // Pen up
T                        // Return home
`Square Complete` /N
;
```

#### Circle Approximation
```mint
// Draw circle using 8-point approximation
:Z
[20 14 0 14 20 14 0 14 20] xpts!   // X coordinates
[0 14 20 14 0 14 20 14 0] ypts!     // Y coordinates

`Drawing Circle...` /N
T                        // Home first
1 pen!                   // Pen down

8(                       // Loop 8 times
  xpts /i ? ypts /i ? U  // Move to next point
)

0 pen!                   // Pen up
T                        // Return home
`Circle Complete` /N
;
```

---

## Practical Programming Examples

### 1. Fibonacci Sequence Generator

```mint
// Generate Fibonacci numbers up to 16-bit limit
:F
n!                       // Get number of terms
0 a! 1 b!               // Initialize first two terms
0 /c!                   // Clear carry flag

`Fibonacci Sequence:` /N

n(                      // Loop n times
  a .                   // Print current term
  32 /C                 // Print space
  
  a b + c!              // Calculate next term
  /c /T = (             // Check for overflow
    `(Overflow)` /N
    0 /c!               // Clear carry
  )
  
  b a!                  // Shift values
  c b!
)
/N
;

// Test: 15 F (generates first 15 Fibonacci numbers)
```

### 2. Prime Number Sieve

```mint
// Simple prime finder using trial division
:P
limit!                   // Get upper limit
`Primes up to ` limit . `: ` /N

2 i!                     // Start with 2
/U(                      // Unlimited loop
  i limit <= /W          // While i <= limit
  
  /T isprime!            // Assume prime
  2 j!                   // Start checking divisors
  
  /U(                    // Check divisors
    j j * i <= /W        // While j² <= i
    i j % 0 = (          // If i divisible by j
      /F isprime!        // Not prime
    )
    j 1 + j!             // Next divisor
  )
  
  isprime /T = (         // If still prime
    i .                  // Print number
    32 /C                // Print space
  )
  
  i 1 + i!               // Next number
)
/N
;

// Test: 30 P (finds all primes up to 30)
```

### 3. Digital Clock Display

```mint
// Simple time display (simulated)
:T
0 h! 0 m! 0 s!          // Initialize hours, minutes, seconds

/U(                     // Infinite loop
  // Display time
  h 10 < ( `0` ) h .    // Hours with leading zero
  `:` 
  m 10 < ( `0` ) m .    // Minutes with leading zero  
  `:`
  s 10 < ( `0` ) s .    // Seconds with leading zero
  /N
  
  // Increment time
  s 1 + s!              // Increment seconds
  s 60 = (              // If 60 seconds
    0 s!                // Reset seconds
    m 1 + m!            // Increment minutes
    m 60 = (            // If 60 minutes
      0 m!              // Reset minutes
      h 1 + h!          // Increment hours
      h 24 = (          // If 24 hours
        0 h!            // Reset hours
      )
    )
  )
  
  100( 100( 100() ) )   // Delay (approximate 1 second)
)
;
```

---

## Hardware Integration Tips

### 1. Motor Driver Connections
- Use ULN2003 or similar driver ICs for stepper motors
- Connect TEC-1 output ports to driver inputs
- Ensure adequate power supply for motors

### 2. End-Stop Switch Setup
- Use normally-open microswitches
- Connect between TEC-1 input ports and ground
- Enable internal pull-up resistors in software

### 3. Pen Mechanism Options
- **Servo Motor**: Most precise, connect to spare output port
- **Solenoid**: Simple on/off action
- **Manual**: User-operated for testing

### 4. Power Considerations
- Separate power supplies for logic and motors recommended
- Use proper current limiting for stepper motors
- Add flyback diodes for inductive load protection

---

## Troubleshooting Guide

### Common Issues

1. **Motors not stepping**: Check driver connections and power
2. **Erratic movement**: Verify timing delays and step sequences
3. **Position drift**: Implement proper homing routine
4. **Buffer overruns**: Keep MINT code lines under 256 characters
5. **Stack underflow**: Ensure proper RPN syntax in MINT

### Debug Techniques

```mint
// Debug position tracking
:D
`Debug - X:` a . ` Y:` b . /N
`Stack depth:` /D . /N
;

// Test motor phases individually
:M
`Testing X motor phases` /N
F E G E H E I E         // Step through phases slowly
`Testing Y motor phases` /N  
J E K E L E M E         // Step through phases slowly
;
```

---

## Conclusion

This guide provides a comprehensive foundation for building an EtchABot controller using the TEC-1 computer. The MINT programming examples are designed to be practical and educational, demonstrating both basic motor control and advanced features like G-code interpretation.

Remember to:
- Test each function individually before integration
- Use proper delays for reliable motor operation
- Implement safety features like end-stop checking
- Keep code modular for easier debugging and maintenance

The combination of MINT's simplicity and the TEC-1's direct hardware access makes this an excellent platform for learning embedded control systems and exploring creative programming solutions.
