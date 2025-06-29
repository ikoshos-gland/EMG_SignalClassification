# Project TODO

## 0. Project Bootstrap
- [ ] Clone / open the repo, verify structure  
- [ ] Install Python libraries: `bleak nest_asyncio opencv-python pandas pytictoc`  
- [ ] Install Arduino / PlatformIO with ESP32-C3 support  
- [ ] Install Arduino libraries: `TensorFlowLite_ESP32`, `Adafruit_MPU6050`, `ESP32_C3_ISR_Servo`

## 1. Hardware Bring-Up
- [ ] Wire 3 MyoWare EMG sensors to ADC pins (ESP32 0, 2, 3)  
- [ ] Wire MPU-6050 to I²C (SDA 9, SCL 8)  
- [ ] Attach SD socket to SPI (MOSI 4, MISO 6, SCK 5, CS 1)  
- [ ] Attach servos  
  - Grip servo → GPIO 5 (`PIN_SERVO_MOV`)  
  - Wrist servo → GPIO 6 (`PIN_SERVO_TURN`)  
- [ ] Format SD card (FAT32) and create directories  
  ```
  /DAT/Task1  /DAT/Task2  /DAT/Task3  /DAT/Task4  /DAT/Task5
  ```
- [ ] Flash [`src/data_collection/integrado_struct.ino`](src/data_collection/integrado_struct.ino:1) and verify BLE advertises “MyESP32”

## 2. Data Acquisition
- [ ] From PC: `cd src/data_collection && python mainö.py`  
- [ ] Select subject and experiment in console prompt  
- [ ] Ensure static images are present in `static/`  
- [ ] Collect ≥30 recordings per task for each participant  
- [ ] After session, back-up `/DAT` folder from SD card

## 3. Feature Extraction
- [ ] Open [`src/feature_extraction/feature_extraction.ipynb`](src/feature_extraction/feature_extraction.ipynb:1)  
- [ ] Implement .dat loader to NumPy structured array  
- [ ] Reproduce real-time pipeline (1 kHz → 50 ms RMS → 2 s window → 12 features)  
- [ ] Save `X.npy` (features) and `y.npy` (labels)

## 4. Model Training & Evaluation
- [ ] Use [`src/train_model/train_test_model.py`](src/train_model/train_test_model.py:1) or notebook  
- [ ] Split train/val/test by subject  
- [ ] Tune model for ≥ 90 % balanced accuracy  
- [ ] Export `saved_model`

## 5. Conversion & Embedding
- [ ] Convert to TFLite and then to C array  
  ```python
  converter = tf.lite.TFLiteConverter.from_saved_model('saved_model')
  converter.optimizations = [tf.lite.Optimize.DEFAULT]
  open('prosthetic_model.tflite','wb').write(converter.convert())
  xxd -i prosthetic_model.tflite > src/real_time/modeldata.cpp
  ```  
- [ ] Update [`modeldata.h`](src/real_time/modeldata.h:1) symbols if needed

## 6. Firmware Build
- [ ] Open [`src/real_time/real_time.ino`](src/real_time/real_time.ino:1), adjust `kTensorArenaSize` if necessary  
- [ ] Compile & flash to ESP32-C3  
- [ ] Check serial monitor for inference outputs

## 7. Benchmark & Calibration
- [ ] Verify loop timing (~2 s) and no watchdog resets  
- [ ] Adjust prediction threshold or servo angles  
- [ ] Fine-tune `OPEN_ANGLE`, `CLOSE_ANGLE`, `TURN_ANGLE` constants

## 8. Field Testing
- [ ] Perform live tests, log serial output and video  
- [ ] Identify misclassifications; gather additional data; iterate steps 3-6

## 9. Documentation & Hand-Off
- [ ] Update `README.md` with wiring diagrams and instructions  
- [ ] Add flashing and running steps  
- [ ] Commit final `modeldata.*` and backup datasets

## 10. Nice-to-Haves / Future Work
- [ ] Auto-create SD directories in firmware  
- [ ] Replace blocking delays with `esp_timer`  
- [ ] Stream live features over BLE for debugging  
- [ ] Switch to sliding 1-s window for lower latency  
- [ ] Implement OTA firmware updates