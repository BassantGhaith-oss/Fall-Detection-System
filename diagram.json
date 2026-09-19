
#Setup + import Libraries
from machine import Pin, I2C ,PWM
from time import sleep_ms, ticks_ms, ticks_diff 
import math 
 
# Pins state defining & numbering
GREEN_LED_PIN = 14 
RED_LED_PIN = 15 
BUZZER_PIN = 16 
RESET_BUTTON_PIN = 17 
 
green_led = Pin(GREEN_LED_PIN, Pin.OUT) 

red_led = Pin(RED_LED_PIN, Pin.OUT) 

#supply the buzzer with varying input(PWM) to make sound frequency

buzzer = PWM(Pin(BUZZER_PIN))
buzzer.freq(2000)  # setting frequency
buzzer.duty_u16(0) # setting duty cycle for the PWM
 
reset_button = Pin( 
    RESET_BUTTON_PIN, 
    Pin.IN, 
    Pin.PULL_UP 
) 
 #Communication Control for the sensor and LCD
 #The components have different adresses so we used same GPIOs without conflict
i2c = I2C( 
    0, 
    scl=Pin(5), 
    sda=Pin(4), 
    freq=400000 
) 
 
# setting sensor address
MPU6050_ADDR = 0x68 
 
# MPU6050 registers 
PWR_MGMT_1 = 0x6B 
ACCEL_CONFIG = 0x1C 
ACCEL_XOUT_H = 0x3B 
 
 
# ±8g range 
# 4096 LSB/g 
ACCEL_SCALE = 4096.0 
 
 
def mpu_write(register, value): 
    i2c.writeto_mem( 
        MPU6050_ADDR, 
        register, 
        bytes([value]) 
    ) 
 
 
def mpu_init(): 
 
    # Wake up MPU6050 
    mpu_write(PWR_MGMT_1, 0x00) 
 
    # Set accelerometer range to ±8g 
    mpu_write(ACCEL_CONFIG, 0x10) 
 
    sleep_ms(100) 
 
 
def read_raw_acceleration(): 
 
    data = i2c.readfrom_mem( 
        MPU6050_ADDR, 
        ACCEL_XOUT_H, 
        6 
    ) 
 
    ax = (data[0] << 8) | data[1] 
    ay = (data[2] << 8) | data[3] 
    az = (data[4] << 8) | data[5] 
 
    # Convert unsigned 16-bit to signed 
    if ax >= 32768: 
        ax -= 65536 
 
    if ay >= 32768: 
        ay -= 65536 
 
    if az >= 32768: 
        az -= 65536 
 
    return ax, ay, az 
 
 
def get_acceleration(): 
 
    ax_raw, ay_raw, az_raw = read_raw_acceleration() 
 
    ax = ax_raw / ACCEL_SCALE 
    ay = ay_raw / ACCEL_SCALE 
    az = az_raw / ACCEL_SCALE 
 
    # Total acceleration magnitude 
    magnitude = math.sqrt( 
        ax * ax + 
        ay * ay + 
        az * az 
    ) 
 
    return magnitude 
 
# LCD 1602 I2C DRIVER 
 
LCD_ADDR = 0x27 
 
LCD_CLEAR = 0x01 
LCD_HOME = 0x02 
 
LCD_ENTRY_MODE = 0x06 
LCD_DISPLAY_ON = 0x0C 
LCD_FUNCTION_SET = 0x28 
 
 
def lcd_write_byte(data): 
    i2c.writeto( 
        LCD_ADDR, 
        bytes([data | 0x08]) 
    ) 
 
 
def lcd_pulse_enable(data): 
    i2c.writeto( 
        LCD_ADDR, 
        bytes([data | 0x0C]) 
    ) 
 
    sleep_ms(1) 
 
    i2c.writeto( 
        LCD_ADDR, 
        bytes([data | 0x08]) 
    ) 
 
    sleep_ms(1) 
 
 
def lcd_write_nibble(nibble, rs): 
 
    data = nibble | 0x08 
 
    if rs: 
        data |= 0x01 
 
    lcd_pulse_enable(data) 
 
 
def lcd_send(value, rs): 
 
    high = value & 0xF0 
    low = (value << 4) & 0xF0 
 
    lcd_write_nibble(high, rs) 
    lcd_write_nibble(low, rs) 
 
 
def lcd_command(command): 
    lcd_send(command, False) 
    sleep_ms(2) 
 
 
def lcd_data(data): 
    lcd_send(data, True) 
 
 
def lcd_init(): 
 
    sleep_ms(50) 
 
    lcd_write_nibble(0x30, False) 
    sleep_ms(5) 
 
    lcd_write_nibble(0x30, False) 
    sleep_ms(1) 
 
    lcd_write_nibble(0x30, False) 
    sleep_ms(1) 
 
    lcd_write_nibble(0x20, False) 
 
    lcd_command(LCD_FUNCTION_SET) 
    lcd_command(LCD_DISPLAY_ON) 
    lcd_command(LCD_ENTRY_MODE) 
    lcd_command(LCD_CLEAR) 
 
    sleep_ms(5) 
 
 
def lcd_clear(): 
    lcd_command(LCD_CLEAR) 
    sleep_ms(2) 
 
 
def lcd_set_cursor(row, col): 
 
    if row == 0: 
        address = 0x80 + col 
    else: 
        address = 0xC0 + col 
 
    lcd_command(address) 
 
 
def lcd_print(text): 
 
    for char in text: 
        lcd_data(ord(char)) 
 
 
def lcd_message(line1, line2=""): 
 
    lcd_clear() 
 
    lcd_set_cursor(0, 0) 
    lcd_print(line1[:16]) 
 
    lcd_set_cursor(1, 0) 
    lcd_print(line2[:16]) 
 
 

# SYSTEM STATES 

 
MONITORING = 0 
POSSIBLE_IMPACT = 1 
CHECKING = 2 
FALL_DETECTED = 3 
 
state = MONITORING 
 

# DETECTION PARAMETERS 
 
# Sudden acceleration threshold 

IMPACT_THRESHOLD = 1.5       # g 
 
# Maximum allowed time after impact 
TIME_WINDOW = 3000           # milliseconds 
 
# Difference from 1g considered low movement 
LOW_MOVEMENT_THRESHOLD = 0.15  # g 
 
# Number of consecutive low movement readings 
LOW_MOVEMENT_REQUIRED = 8 
 
# VARIABLES 
 
impact_time = 0 
low_movement_count = 0 
 
previous_acceleration = 1.0 
 
 

# STATE FUNCTIONS 

 
def set_monitoring(): 
 
    global state 
    global low_movement_count 
 
    state = MONITORING 
    low_movement_count = 0 
 
    green_led.value(1) 
    red_led.value(0) 
    buzzer.duty_u16(0)
 
    lcd_message( 
        "Monitoring...", 
        "System Ready" 
    ) 
 
 
def set_possible_impact(): 
 
    global state 
    global impact_time 
    global low_movement_count 
 
    state = POSSIBLE_IMPACT 
 
    # Start the time window exactly when impact happens 
    impact_time = ticks_ms() 
 
    low_movement_count = 0 
 
    green_led.value(1) 
    red_led.value(0) 
    buzzer.duty_u16(0)
 
    lcd_message( 
        "Possible Impact!", 
        "Checking..." 
    ) 
 
 
def set_checking(): 
 
    global state 
 
    state = CHECKING 
 
    green_led.value(1) 
    red_led.value(0) 
    buzzer.duty_u16(0) 
 
    lcd_message( 
        "Checking...", 
        "Movement..." 
    ) 
 
 
def set_fall_detected(): 
 
    global state 
 
    state = FALL_DETECTED 
 
    green_led.value(0) 
    red_led.value(1) 
    buzzer.duty_u16(32768)
    lcd_message( 
        "FALL DETECTED!", 
        "Press RESET" 
    ) 
 
# INITIALIZATION 
 
mpu_init() 
lcd_init() 
 
set_monitoring() 
 
# MAIN LOOP 

while True: 
 
    # RESET BUTTON 
   
    if reset_button.value() == 0: 
 
        sleep_ms(50) 
 
        if reset_button.value() == 0: 
 
            set_monitoring() 
 
            # Wait until button released 
            while reset_button.value() == 0: 
                sleep_ms(10) 
 
 
   
    # READ ACCELERATION 

 
    acceleration = get_acceleration() 
 
 
   
    # MONITORING 

 
    if state == MONITORING: 
 
        # Sudden impact detected 
        if acceleration >= IMPACT_THRESHOLD: 
 
            set_possible_impact() 
 
            # Immediately go to checking 
            set_checking() 
 
 
    # CHECKING 
    
 
    elif state == CHECKING: 
 
        # Calculate how much time passed since impact 
        elapsed = ticks_diff( 
            ticks_ms(), 
            impact_time 
        ) 
 
        # Check for low movement 
        movement_difference = abs( 
            acceleration - 1.0 
        ) 
 
        if movement_difference <= LOW_MOVEMENT_THRESHOLD: 
 
            low_movement_count += 1 
 
        else: 
 
            low_movement_count = 0 
 
         # FALL CONFIRMATION 
        
        if low_movement_count >= LOW_MOVEMENT_REQUIRED: 
 
            set_fall_detected() 

        # FALSE ALARM      
 
        elif elapsed >= TIME_WINDOW: 
 
            set_monitoring() 
 
  
    #  FALL DETECTED 
 
    elif state == FALL_DETECTED: 
 
        buzzer.duty_u16(32768) 
        red_led.value(1) 
        green_led.value(0) 
 
 
    # Small delay 
    sleep_ms(50)
