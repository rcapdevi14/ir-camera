# ir-camera
Project IR mini camera to the laptop

Components were bought in adafruit.com

1) Adafruit MLX90640 IR Thermal Camera Breakout - 55 Degree. Product ID: 4407 ($74.95)
2) Adafruit MCP2221A Breakout - General Purpose USB to GPIO ADC I2C - Stemma QT / Qwiic. Product ID: 4471 ($6.50)
3) USB C to USB C Cable - USB 3.1 Gen 4 with E-Mark - 1 meter long. Product ID: 4199 ($9.95)
4) STEMMA QT / Qwiic JST SH 4-Pin Cable - 50mm Long. Product ID: 4399 ($0.95)
5) White Nylon Machine Screw and Stand-off Set – M2.5 Thread. Product ID: 3658 ($14.95)
6) Adafruit Swirly Aluminum Mounting Grid for 0.1" Spaced PCBs - 5x5. Product ID: 5779 ($3.95)

Pictures are attached.

I am running Python locally with anaconda. Run the following commands

- conda create -n IRcamera python=3.10 -y   #create a new environment
- conda activate IRcamera
- export PATH="/Users/rodolfocapdevilla/opt/anaconda3/envs/IRcamera_v2/bin:$PATH"
- pip install hidapi adafruit-blinka adafruit-circuitpython-mlx90640 numpy matplotlib

This is the python script:

import os

# Set environment variable for Blinka to use MCP2221 (must be before importing board or busio)
os.environ["BLINKA_MCP2221"] = "1"

import time
import numpy as np
import matplotlib.pyplot as plt
import board
import busio
import adafruit_mlx90640

# Set environment variable for Blinka to use MCP2221
os.environ["BLINKA_MCP2221"] = "1"

# Initialize I2C bus (increase frequency for smoother performance)
i2c = busio.I2C(board.SCL, board.SDA, frequency=800000)

# Initialize the MLX90640 sensor
mlx = adafruit_mlx90640.MLX90640(i2c)
print("MLX90640 detected on I2C!")

# Set refresh rate (options: REFRESH_0_5_HZ, REFRESH_1_HZ, REFRESH_2_HZ, up to REFRESH_32_HZ; higher rates may need testing for stability)
mlx.refresh_rate = adafruit_mlx90640.RefreshRate.REFRESH_2_HZ

# Buffer for frame data (32x24 pixels = 768 values)
frame = np.zeros(768)

# Enable interactive mode for live updates
plt.ion()

# Main loop for live display
while True:
    try:
        # Read a frame of data
        mlx.getFrame(frame)
    except ValueError:
        # Skip bad frames
        continue

    # Reshape into 24x32 grid (height x width)
    data = np.reshape(frame, (24, 32))

    # Display as heatmap with interpolation for better visuals
    plt.imshow(data, cmap='inferno', interpolation='bicubic')
    plt.colorbar()
    plt.title("Live Thermal Image (Press Ctrl+C to exit)")

    # Update the plot
    plt.draw()
    plt.pause(0.05)  # Adjust pause for refresh rate (smaller = faster updates)

    # Clear for next frame
    plt.clf()


