# AT90 Microcontroller Lab Buttons

/*
Lab Week 3: Buttons and LEDs

Read the instructions about how the lab is organised on Canvas under Assessments:

Preparation: Try out the given examples on the lecture slides

Study the lecture slides “ENEL608 Lecture 3b Buttons buzzer lamp”. Try out some of the examples.

Discuss what you don’t understand with peers in the lab and with the lab supervisor.

You will learn the following useful skills that are directly related to this lab:
• How to check if a button is pressed
• How to check if a button is pressed and then released again
• Overall structure of a microcontroller program, as well as functions and macros

Your preparatory work does not need to be submitted on Canvas.

Lab Exercise: Air conditioning control panel

Design the state diagram and write the C code for the control panel of an air conditioning system. The
control panel works as follows:

The control panel displays either the temperature or the fan speed. Button 2 (PA2) toggles between the
the modes.
• Temperature
o The temperature range is 20-29 degrees.
o The current temperature is displayed on the LED number display which is connected to
PORTC.
o This example picture shows a temperature of 23 degrees. Note that some of the eight

LEDs below the number display also light up, because they are connected to the same
port.
• Fan speed
o The speed ranges from 1 to 3.
o The speed is displayed by the LED strip below the number display as follows:
 Speed 1 = PC0 on, PC1:7 off
 Speed 2 = PC0 on, PC1 on, PC2:7 off
 Speed 3 = PC0 on, PC1 on, PC2 on, PC3:7 off

Introduction to Microcontrollers

o Example display for speed 2:

Remarks: On my board, the LED strip happens to be mirrored. So, what is labelled as PC7
is actually PC0. Some lab boards in WS413 also have mirrored LED strips. (You can see
that the number display above shows the value 03, because it is also connected Port C, as
mentioned before.)
Button 1 (PA1) is the “up button”.

• It increases the currently displayed value (either temperature or fan speed, depending on the
current display mode) by one step.
Button 0 (PA0) is the “down button”.

• It decreases the currently displayed value (either temperature or fan speed, depending on the
current display mode) by one step.
These are the buttons to be used:

Hints:
• Start with the state diagram.
• Use a function to check if a button has been pressed and also released. Otherwise, your program
will be difficult to use.
• The problem can be solved by about 100 lines of C code.
• It may help to write dedicated functions for displaying the temperature and the fan speed.
• Note that the first digit of the temperature is always two. So you will only need to change the
lower 4 bits of Port C, whereas the upper 4 bits can remain the same.
*/

/*
*	GccApplication1.c
 *
*	Created: 20/03/2025 3:00:38 pm
*	Author : 
 */ 
/*Preprocessor Directives

#define F_CPU 8000000L → Defines the CPU clock speed at 8 MHz, necessary for the delay functions.

#include <avr/io.h> → Allows interaction with the ATmega microcontroller's I/O registers.

#include <util/delay.h> → Provides _delay_ms() for timing delays.

*/

#define F_CPU 8000000L

#include <avr/io.h>

#include <util/delay.h>

// Define button pins

/*Button & Range Definitions

*#define DOWN_BUTTON PA0, #define UP_BUTTON PA1, #define MODE_BUTTON PA2 *These define button pins for down (PA0), up (PA1), and mode toggle (PA2).

*/

#define DOWN_BUTTON PA0  // PA0 (Button 0) - Decreases temperature or fan speed

#define UP_BUTTON PA1    // PA1 (Button 1) - Increases temperature or fan speed

#define MODE_BUTTON PA2  // PA2 (Button 2) - Toggles between temperature and fan speed modes

// Define range limits

//*Temperature range (MIN_TEMP = 20, MAX_TEMP = 29) and fan speed range (MIN_FAN_SPEED = 1, MAX_FAN_SPEED = 3).

#define MIN_TEMP 20

#define MAX_TEMP 29

#define MIN_FAN_SPEED 1

#define MAX_FAN_SPEED 3

// Function prototypes

/*Function Prototypes

*Declares helper functions before the main code:

*initializePorts() → Configures I/O.

*checkButtonPress() → Detects button press and release. *displayTemperature() → Displays temperature on PORTC.

displayFanSpeed() → Displays fan speed on PORTC
*/
void initializePorts(void);

uint8_t checkButtonPress(uint8_t button);

void displayTemperature(uint8_t temp); void displayFanSpeed(uint8_t speed);

// Global variables

/*uint8_t currentTemp = 23; → Default temperature (23°C). uint8_t currentFanSpeed = 1; → Default fan speed (1).
uint8_t displayMode = 0; → 0 = Temperature mode, 1 = Fan speed mode
*/
uint8_t currentTemp = 23;      // Default temperature uint8_t currentFanSpeed = 1;   // Default fan speed uint8_t displayMode = 0;       // 0 = Temperature, 1 = Fan Speed

/*Main Program (main)

Calls initializePorts(); → Sets up button inputs and LED outputs.
Displays default temperature on LEDs (displayTemperature(currentTemp))

*/ int main(void) {
    // Initialize ports     initializePorts();
    
    // Initial display
    displayTemperature(currentTemp);
/*

Loop (while (1))

The loop continuously:

Toggles Display Mode (PA2)

Switches between temperature and fan speed when Mode Button is pressed.

Updates the LED display accordingly.

Increases the value (PA1 - Up Button)

If in temperature mode, increments the temperature up to MAX_TEMP.

If in fan speed mode, increments fan speed up to MAX_FAN_SPEED.

Decreases the value (PA0 - Down Button)

If in temperature mode, decreases the temperature down to MIN_TEMP.

If in fan speed mode, decreases fan speed down to MIN_FAN_SPEED.

Delay (_delay_ms(50)) → Helps prevent button bouncing.

*/        while (1) {
        // Check mode button (PA2) - Toggles between temperature and fan speed display         if (checkButtonPress(MODE_BUTTON)) {
            displayMode = !displayMode;  // Toggle between temperature and fan speed
            
            // Update display based on new mode             if (displayMode == 0) {                 displayTemperature(currentTemp);
            } else {
                displayFanSpeed(currentFanSpeed);
            }
        }
        
        // Check up button (PA1) - Increases the current value (temperature or fan speed)         if (checkButtonPress(UP_BUTTON)) {             if (displayMode == 0) {  // Temperature mode                 if (currentTemp < MAX_TEMP) {                     currentTemp++;
                    displayTemperature(currentTemp);
                }
            } else {  // Fan speed mode
                if (currentFanSpeed < MAX_FAN_SPEED) {                     currentFanSpeed++;
                    displayFanSpeed(currentFanSpeed);
                }
            }
        }
        
        // Check down button (PA0) - Decreases the current value (temperature or fan speed)         if (checkButtonPress(DOWN_BUTTON)) {             if (displayMode == 0) {  // Temperature mode                 if (currentTemp > MIN_TEMP) {                     currentTemp--;
                    displayTemperature(currentTemp);
                }
            } else {  // Fan speed mode
                if (currentFanSpeed > MIN_FAN_SPEED) {                     currentFanSpeed--;
                    displayFanSpeed(currentFanSpeed);
                }
            }
        }
        
        _delay_ms(50);  // Increased delay to prevent button bouncing
    }
    
    return 0; }

// Initialize ports void initializePorts(void) {
    // Set PORTA as input for buttons (with pull-up resistors)
    // PA0: Down button, PA1: Up button, PA2: Mode button
    DDRA &= ~((1 << DOWN_BUTTON) | (1 << UP_BUTTON) | (1 << MODE_BUTTON));
    PORTA |= (1 << DOWN_BUTTON) | (1 << UP_BUTTON) | (1 << MODE_BUTTON);
    
    // Set PORTC as output for LED display
    DDRC = 0xFF;  // All pins as output
    PORTC = 0x00; // All LEDs initially off
    
    // Initialize with default temperature display     displayTemperature(currentTemp);
}

// Check if a button is pressed and released

// button: The button pin to check (PA0, PA1, or PA2)

uint8_t checkButtonPress(uint8_t button) {
    if (!(PINA & (1 << button))) {  // Button is pressed (active low)
        _delay_ms(50);  // Increased debounce delay
        
        if (!(PINA & (1 << button))) {  // Still pressed after debounce
            // Wait for button release             while (!(PINA & (1 << button))) {
                _delay_ms(10);
            }
            _delay_ms(50);  // Increased debounce delay for release             return 1;  // Button was pressed and released
        }
    }
    return 0;  // Button was not pressed
}

// Display temperature on LED display void displayTemperature(uint8_t temp) {     // Ensure temperature is within valid range     if (temp < MIN_TEMP) temp = MIN_TEMP;     if (temp > MAX_TEMP) temp = MAX_TEMP;
    
    // Clear the display first
    PORTC = 0x00;
    
    // For temperature, display the digit in 7-segment format     // For this simple display, we'll just show the number directly     uint8_t upperBits = 0x20;  // Represent digit 2 in upper 4 bits     uint8_t lowerBits = temp - MIN_TEMP;  // Adjust to get the second digit (0-9)
    
    // Combine and display
    PORTC = upperBits | lowerBits;
}

// Display fan speed on LED strip void displayFanSpeed(uint8_t speed) {     // Ensure fan speed is within valid range     if (speed < MIN_FAN_SPEED) speed = MIN_FAN_SPEED;     if (speed > MAX_FAN_SPEED) speed = MAX_FAN_SPEED;
    
    // Clear all LEDs first
    PORTC = 0x00;
    
    // Create the fan speed digit display for the number display portion
    // This will display the actual speed number (1, 2, or 3)
    PORTC = speed;
    
    // Set appropriate LEDs for the fan speed PATTERN according to lab requirements     switch (speed) {         case 1:
            // Speed 1 = PC0 on, PC1:7 off             PORTC = 0x01;             break;         case 2:
            // Speed 2 = PC0 on, PC1 on, PC2:7 off             PORTC = 0x03;             break;         case 3:
            // Speed 3 = PC0 on, PC1 on, PC2 on, PC3:7 off             PORTC = 0x07;             break;         default:
            PORTC = 0x00;             break;
    }
}

