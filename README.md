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
- export PATH="/Users/rodolfocapdevilla/opt/anaconda3/envs/IRcamera_v2/bin:$PATH"   #need every time (check "which pip" and "python --version")
- pip install hidapi adafruit-blinka adafruit-circuitpython-mlx90640 numpy matplotlib
- pip install --upgrade --force-reinstall adafruit-blinka adafruit-platformdetect   #if needed
- export BLINKA_MCP2221_RESET_DELAY=0.5
- pip install scipy   #to enhance the resolution, if needed

#This is the python script:

import os

# Set environment variable for Blinka to use MCP2221 (must be before importing board or busio)
os.environ["BLINKA_MCP2221"] = "1"

import time
import numpy as np
import matplotlib.pyplot as plt
import board
import busio
import adafruit_mlx90640
from scipy.ndimage import zoom

# Initialize I2C bus (increase frequency for smoother performance)
i2c = busio.I2C(board.SCL, board.SDA, frequency=400000)

# Initialize the MLX90640 sensor
mlx = adafruit_mlx90640.MLX90640(i2c)
#mlx.setMode(adafruit_mlx90640.INTERLEAVED_MODE)  # Switch from default CHESS mode
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
        invalid_count = np.sum(frame == -273.15)
        print("Invalid pixels:", invalid_count, "/ 768")
        print("Min temp:", np.min(frame))
        print("Max temp:", np.max(frame))
        print("Sample pixel:", frame[0])  # First pixel value
    except ValueError:
        # Skip bad frames
        continue

# Reshape into 24x32 grid (height x width)
data = np.reshape(frame, (24, 32))
# Handle invalid pixels by replacing with mean of valid temps
invalid_mask = (frame == -273.15)
if np.any(invalid_mask):
    valid_mean = np.mean(frame[~invalid_mask])
    frame[invalid_mask] = valid_mean
# Then reshape (after fixing frame)
data = np.reshape(frame, (24, 32))

# Upscale the image for better resolution (4x factor; adjust as needed, e.g., 2 for smaller)
upscale_factor = 4
data = zoom(data, upscale_factor, order=3)  # Cubic interpolation

# Auto-scale to valid range
valid_data = frame[frame != -273.15]
if valid_data.size > 0:
    vmin = np.min(valid_data)
    vmax = np.max(valid_data)
else:
    vmin, vmax = 0, 50  # Fallback

# Display as heatmap with interpolation for better visuals
plt.imshow(data, cmap='inferno', interpolation='bicubic', vmin=vmin, vmax=vmax)
plt.colorbar()
plt.title("Live Thermal Image (Press Ctrl+C to exit)")

# Update the plot
plt.draw()
plt.pause(0.05)  # Adjust pause for refresh rate (smaller = faster updates)

# Clear for next frame
plt.clf()


