# MyoMod Armband – Open-Source 6-Channel EMG Interface

The **MyoMod Armband** is a modular, open-source **6-channel EMG (electromyography) armband** designed for real-time muscle signal acquisition of the forearm muscles near the ellbow and transmission of the measured signals via Bluetooth Low Energy (BLE). 

![](images/anim_cropped2.gif)

It's general purpose approach enables flexible use in human-machine interfaces, robotics, prostetics and research applications. It is part of the modular **MyoMod architecture** and as such the firmware of the armband is easily customizeable. This also allows wired connections to external devices via MyoMod Link.

---

## 🧩 Architecture Overview

The Data Processing Unit (DPU) is the brain of the Armband and orchestrates all data transmission. It is based on the 600 MHz NXP i.MX RT 1062 microcontroller and as such is able to perform all the needed digital signal processing in real time and still has a lot of computational capacity left for other tasks. 

Apart from the DPU the ESP32-S3 based BLE Bridge and the ADC are the most important components on the Armband. While the ADC captures the raw EMG signals that are to be filtered by the DPU, the BLE-Bridge implements a BLE Device and therefore is the interface to your Webapp, VR-game, etc. The BLE-Bridge is connected to the DPU via MyoMod-Link (a modular, real-time protocol based on I²C) and an async UART-based protocoll for configuration of the DPU (and for later Firmware Upgrades). 

In it's default configuration the Armband transmits the filtered as well as the raw EMG data to the BLE host, so that you can either work directly with the EMG data or implement your own filter algorithm on the host device.

![Architecture Diagram](images/architecture.drawio.png)

Next to these key devices the Armband also includes
* a 6 dof inertial measurement unit (IMU) that can further support the analysis of the measured EMG signals
* a battery management system (BMS) for charging the Armband via USB-C
* the user interface consisting of a button and a RGB-LED 
* and last but no least a socket for connecting an external MyoMod-Link device thus allowing an easy extension of the capabilites of the armband or an usage in an entirely new area.

---

## 🧠 (Web)BLE API

The MyoMod Armband communicates with host devices via **Bluetooth Low Energy (BLE)** using a custom GATT service. The API provides real-time access to EMG data and device configuration.

### BLE Service & Characteristics

**Primary Service UUID:** `f1f1d764-f9dc-4274-9f59-325fea6d631b`

| Characteristic | UUID | Type | Description |
|---------------|------|------|-------------|
| **Raw EMG** | `9c54ed76-847e-4d51-84be-7cf02794de53` | Notify | Raw 6-channel EMG signals (15 samples/channel) |
| **Filtered EMG** | `36845417-f01b-4167-afa1-81b322238fe1` | Notify | Processed EMG data (6 channels + state) |
| **Combined EMG** | `ab92c948-16ed-4ed2-b18e-d4c0a27808fc` | Notify | Raw + filtered EMG in single packet |
| **DPU Control** | `5f2d5b5b-2166-4d71-9b4a-ea719ce9777e` | Write/Notify | Device configuration and control protocol |

### Data Structures


**EMG Data** (15 samples per channel):
```typescript
type MyoModEmgData = {
  chnA: Float32Array;  // Channel A samples
  chnB: Float32Array;  // Channel B samples  
  chnC: Float32Array;  // Channel C samples
  chnD: Float32Array;  // Channel D samples
  chnE: Float32Array;  // Channel E samples
  chnF: Float32Array;  // Channel F samples
};
```

**Filtered EMG Data**:
```typescript
type MyoModFilteredEmgData = {
  data: Float32Array;  // 6 processed channel values
  state: number;       // Processing state indicator
};
```

**Combined EMG Data**:
```typescript
type MyoModCombinedEmgData = {
  raw: MyoModEmgData;           // Raw EMG data (6 channels × 15 samples)
  filtered: MyoModFilteredEmgData;  // Filtered EMG data (6 channels + state)
};
```

**DPU Control Protocol**:

The async DPU control protocol is text based and is documented here: [MyoMod WebApp repository](#).

### WebBLE Implementation

A comprehensive React SDK is available with full API documentation, including connection management, data subscription, device configuration, and error handling. See the [MyoMod WebApp repository](#) for complete implementation details and examples.

**Browser Support**: Chrome/Edge 56+, Safari 16.4+. Requires HTTPS for WebBLE access.

### Unreal Engine Implementation
There is an exsiting implementation for real time transmission of the EMG data via BLE for the android plttform in unreal engine. ATM we are discussing if it can be open-sourced due to the usage of a plugin used for BLE interaction.

---

## ⚙️ Hardware Overview


Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam

![Hardware Diagram Placeholder](docs/architecture-diagram.png)

Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.

---

## 🧩 Integration into MyoMod ecosystem

The MyoMod Armband is part of the broader **MyoMod ecosystem**, which enables modular and open development of prosthetic and biosignal systems. The ecosystem is built around the **Data Processing Unit (DPU)** that orchestrates communication between various input and output devices through the standardized **MyoMod-Link protocol**.


### Modular Architecture

The MyoMod ecosystem follows a **node-based abstraction model** where each sensor or actuator is represented as individual nodes that have ports for data interconnection and can be parametrized. This allows algorithms to be developed independently of the specific hardware configuration:

There are different types of **nodes**, seperated by their function.
- **Devices**: Devices connected to the DPU via MyoMod-Link, they can be inputs, outputs or both. Examples are EMG sensors, IMUs, force sensors, Prosthetic hands, robotic actuators, feedback systems or the BLE Bridge. Also Bridges to other interfaces like SPI or UART could be devices.
- **Embedded Devices**: The same as normal devices, but integrated into the DPU, e.g. the DPU of the Armband is directly connected to the ADC, but the ADC is still handled as a virtual or embedded device for the configuration.
- **Algorithms**: Algorithms are nodes without connection to the external world. This could for example be algorithms for real-time signal processing, machine learning inference or simple linear functions.

Devices can be connected to each other by connecting their **ports**. Ports have a defined data-type and only input and output-ports of the same type can be connected. This allows powerfull indivudalisation of system in the MyoMod ecosystem by non-programmers, as they only need to parametrize and connect different nodes in a visual configuration environment.

![Node Diagram Placeholder](images/configurationEditor.png)

### Configuration Management

A configuration describes all used nodes, their parameters and theri interconnection. The system supports **multiple configurations** that can be switched at runtime, enabling for example task-specific optimization with different algorithms for precise vs. power focused use-cases with the same Hardware. 

MyoMod Link allows automatic detection of all connected devices, this in turn allows an automatic selection of
configurations based on these connected devices. For prostetics use-cases this allows swapping the actual prostetic hand and directly adapting the control algorithm to it. For the Armband this allows different behaviour depending on the connected device at the expansion port, e.g. a connected Display can show the muscle contraction in real-time or a connected audio device produces sounds depending on the contracted muscles.


### MyoMod-Link Protocol

MyoMod-Link is a **modular, real-time protocol based on I²C** that provides:

- **100Hz cyclic communication** for real-time EMG and control data
- **Asynchronous register-based communication** for device configuration and status
- **Plug-and-play device discovery** with automatic device identification
- **Pipeline architecture** enabling parallel data processing and transmission
- **Multi-port support** for bandwidth scaling across multiple I²C buses on DPU side


### Ecosystem Benefits

- **Modularity**: Easy integration of new sensors and actuators
- **Scalability**: From simple 2-channel systems to complex multi-device setups
- **Flexibility**: Runtime configuration switching for different use cases
- **Development**: Simplified algorithm development independent of hardware
- **Accessibility**: Cost-effective scaling based on user requirements

---

## 🧩 Digital processing of the EMG signals

The MyoMod Armband employs **digital** signal processing to overcome the limitations and costs of traditional analog EMG filtering systems. It does so by using a high resolution, low-noise Sigma-Delta ADC (MAX11254). The usage of a Sigma-Delta ADC greatly reduces the requeirements of the input anti-aliasing filters, as the digitzing frequency is severeal magnitudes higher than the actual sampling rate of the 24 Bit signal. This allows us to move the actual filtering to the digital domain and thus remove the high costs of precision analog filters otherwise needed for this weak EMG signals. 

![Digital Processing Flowchart](images/bewertungVar/BewertungVariableStoerung_eng.svg)

**Adaptive Processing:**
The adaptive EMG filter is able to automatically adapt to interference by analysing the signal in the freqency domain. For this a 512-point FFT with an update rate of 100 Hz is used. This approach also allows for a consistent 0-1 output scaling across environments, is robust against many electronic device emissions (like monitors or microwaves) and also automatically handles 50/60Hz power line interference. Last but no least focusing on digital filtering allows for an easi replacement or refinement of this filter.

---

## 🔗 Related Repositories

| Component | Description | Repository |
|------------|--------------|-------------|
| **MyoMod-Armband-Hardware** | CAD, PCB, and design files for the EMG armband | [Repository link placeholder](#) |
| **MyoMod-Armband-Firmware** | Firmware for EMG acquisition, BLE communication, and preprocessing | [Repository link placeholder](#) |
| **MyoMod-Documentation** | Detailed documentation, setup instructions, and datasheets | [Repository link placeholder](#) |
| **MyoMod-Protocol** | Definition of BLE services and data packet formats | [Repository link placeholder](#) |
| **MyoMod-WebApp** | Web interface for real-time EMG visualization and calibration | [Repository link placeholder](#) |
| **MyoMod-Link** | Modular converter for UART, SPI, I²C, and other communication interfaces | [Repository link placeholder](#) |

---

## 🚀 Getting Started

### Cloning the repository
```bash
git clone https://github.com/MyoMod/MyoMod-Armband.git
cd MyoMod-Armband
