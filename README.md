# IVI: In-Vehicle Infotainment System

## Conceptual Architecture of an IVI System Integrating Media, Navigation and Smartphone Projection

A conceptual design study of an **In-Vehicle Infotainment (IVI)** system
that integrates **media playback, navigation, Android Auto and Apple
CarPlay-style smartphone projection** using a layered Android Automotive
architecture.

The project explains how user input travels through the infotainment
software stack to the hardware, and how sensor and vehicle data travel
back to the driver. It also documents the interfaces, protocols, APIs,
and end-to-end data flows between system components.

> **Project Type:** Conceptual architecture and system design study\
> **Reference Architecture:** Android Automotive concepts\
> **Implementation:** Design/modeling only; no real vehicle hardware or
> production source code

------------------------------------------------------------------------

## 📌 Project Overview

An **In-Vehicle Infotainment (IVI)** system is the combined hardware and
software platform located in a vehicle's dashboard. It provides features
such as:

-   Media playback
-   Navigation
-   Smartphone integration
-   Voice control
-   Hands-free interaction
-   Vehicle information and selected vehicle controls

This project focuses on designing a conceptual IVI architecture that
connects three major functions:

1.  **Media Playback**
2.  **Navigation**
3.  **Smartphone Projection**

The architecture is organized into six layers so that applications
remain separated from vendor-specific hardware implementation.

------------------------------------------------------------------------

## 🎯 Objectives

The main objectives of this project are:

-   Identify the major components required for media, navigation, and
    projection.
-   Organize the components into a clear layered architecture.
-   Define and annotate component interfaces.
-   Identify the protocol, API, or bus used by each interface.
-   Describe the data exchanged between components.
-   Trace the end-to-end data flow for media, navigation, and
    projection.
-   Verify the architecture against the project requirements.
-   Demonstrate how navigation audio can interact with music playback
    through audio focus.

------------------------------------------------------------------------

## 🏗️ System Architecture

The system follows a six-layer conceptual architecture:

``` text
┌─────────────────────────────────────┐
│              Driver                 │
│ Touchscreen | Microphone | Keys     │
└──────────────────┬──────────────────┘
                   │
┌──────────────────▼──────────────────┐
│               Apps                  │
│ Media App | Navigation | Projection │
└──────────────────┬──────────────────┘
                   │ Binder / AIDL
┌──────────────────▼──────────────────┐
│             Services                │
│ Media | Location | Projection       │
└──────────────────┬──────────────────┘
                   │
┌──────────────────▼──────────────────┐
│            Car Layer                │
│ Car Audio Service | Car Property    │
└──────────────────┬──────────────────┘
                   │
┌──────────────────▼──────────────────┐
│               HAL                   │
│ Audio HAL | GNSS HAL | Vehicle HAL  │
└──────────────────┬──────────────────┘
                   │
┌──────────────────▼──────────────────┐
│             Hardware                │
│ Speakers | GPS | CAN/ECUs | Phone   │
└─────────────────────────────────────┘
```

### Layer Responsibilities

  -----------------------------------------------------------------------
  Layer                   Main Components         Responsibility
  ----------------------- ----------------------- -----------------------
  Driver                  Touchscreen,            Provides input and
                          microphone,             receives audio/visual
                          steering-wheel keys     output

  Apps                    Media, Navigation,      Provides user-facing
                          Projection              functionality

  Services                Media, Location,        Manages
                          Projection services     application-level
                                                  operations

  Car Layer               Car Audio Service, Car  Handles
                          Property Service        automotive-specific
                                                  audio and vehicle data

  HAL                     Audio HAL, GNSS HAL,    Provides standardized
                          Vehicle HAL             hardware interfaces

  Hardware                Speakers, GPS receiver, Physical devices and
                          CAN/ECUs, smartphone    external systems
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 🔌 Component Interfaces

The project defines **13 component-to-component interfaces**.

  ----------------------------------------------------------------------------------
  ID                Connection           Interface / Protocol      Main Data
  ----------------- -------------------- ------------------------- -----------------
  I1                Driver ↔ Apps        Linux input subsystem /   Touch, keys,
                                         Android input framework   voice commands,
                                                                   screen and sound
                                                                   output

  I2                Media App ↔ Media    Binder IPC, MediaSession  Play, pause,
                    Service              / MediaBrowser APIs       skip, metadata,
                                                                   playback state

  I3                Navigation App ↔     Binder IPC,               Location requests
                    Location Service     LocationManager API       and updates

  I4                Projection App ↔     Binder IPC, car           Video, audio,
                    Projection Service   projection APIs           touch events,
                                                                   session state

  I5                Media Service ↔ Car  AudioManager / Car API    Audio focus and
                    Audio Service                                  volume

  I6                Projection Service ↔ Binder IPC, Car API       Driving state,
                    Car Property Service                           speed, gear

  I7                Car Audio Service ↔  HAL interface             Routing, volume,
                    Audio HAL                                      PCM audio

  I8                Location Service ↔   HAL interface             Location fixes
                    GNSS HAL                                       and satellite
                                                                   status

  I9                Car Property Service Vehicle HAL interface     Vehicle
                    ↔ Vehicle HAL                                  properties such
                                                                   as speed and gear

  I10               Audio HAL ↔          ALSA / TinyALSA, I2S/TDM  Digital audio
                    Speakers/Amplifier                             samples and
                                                                   controls

  I11               GNSS HAL ↔ GPS       UART / SPI / SoC          Position data
                    Receiver                                       

  I12               Vehicle HAL ↔        CAN through               Vehicle signals
                    CAN/ECUs             gateway/microcontroller   and status

  I13               Projection Service ↔ USB / Wi-Fi with          Video, audio,
                    Smartphone           Bluetooth setup           touch and vehicle
                                                                   data
  ----------------------------------------------------------------------------------

------------------------------------------------------------------------

## 🔄 Data Flows

The project traces three complete end-to-end data flows.

### 1. Media Playback Flow

``` text
Driver
  ↓
Media App
  ↓
Media Service
  ↓
Car Audio Service
  ↓
Audio HAL
  ↓
Speakers / Amplifier
```

#### Steps

-   **A1:** Driver presses Play.
-   **A2:** Media App sends the play command to Media Service.
-   **A3:** Media Service requests audio focus from Car Audio Service.
-   **A4:** Car Audio Service determines routing and volume.
-   **A5:** Audio HAL sends digital audio samples to the
    speakers/amplifier.

------------------------------------------------------------------------

### 2. Navigation Flow

``` text
GPS Receiver
  ↓
GNSS HAL
  ↓
Location Service
  ↓
Navigation App
  ↓
Driver
```

#### Steps

-   **B1:** GPS receiver provides position data to GNSS HAL.
-   **B2:** GNSS HAL provides the location fix to Location Service.
-   **B3:** Location Service sends location updates to Navigation App.
-   **B4:** Navigation App displays the route and provides voice
    guidance.

Navigation voice guidance uses the automotive audio path and audio focus
mechanism.

------------------------------------------------------------------------

### 3. Smartphone Projection Flow

``` text
Smartphone
  ↓
Projection Service
  ↓
Projection App
  ↓
Car Display / Driver
```

#### Steps

-   **C1:** Smartphone sends video, audio and session information.
-   **C2:** Projection Service sends the stream to Projection App.
-   **C3:** Projection App displays the phone interface on the car
    display.

Touch input can travel in the reverse direction from the car display
back to the smartphone.

------------------------------------------------------------------------

## 🎵 Combined Scenario: Navigation Prompt During Music

One important scenario demonstrates how different IVI functions
interact.

``` text
Music Playing
     ↓
Navigation receives location
     ↓
Navigation needs to speak
     ↓
Navigation requests audio focus
     ↓
Car Audio Service applies focus rules
     ↓
Music volume is reduced
     ↓
Navigation prompt plays
     ↓
Prompt ends
     ↓
Music volume is restored
```

The **Car Audio Service** acts as the central point for audio-focus
decisions. In a typical configuration, music is temporarily **ducked**
while a navigation prompt is played.

The exact behavior can vary by vehicle manufacturer.

------------------------------------------------------------------------

## 🚗 Vehicle Data Path

Vehicle information travels from the vehicle network toward
applications:

``` text
CAN Bus / ECUs
      ↓
Vehicle HAL
      ↓
Car Property Service
      ↓
Projection Service / Other Apps
```

Example vehicle data includes:

-   Vehicle speed
-   Gear selection
-   Other supported vehicle properties

This information can be used to limit distracting features while the
vehicle is moving.

------------------------------------------------------------------------

## 🧩 Design Rules

The architecture follows several important rules:

1.  Applications communicate with framework services through
    **Binder/AIDL**.
2.  Applications do not directly communicate with HALs or hardware.
3.  Framework and car services access hardware through HAL interfaces.
4.  Vendor-specific hardware implementation is isolated inside the
    HAL/driver layers.
5.  Vehicle signals reach applications through the **Car Property
    Service**.
6.  Audio focus is managed through the **Car Audio Service**.

These rules help provide hardware independence, safety controls, and
maintainability.

------------------------------------------------------------------------

## 🛠️ Tools & Environment

This project is a **conceptual design and modeling study**, so no
special runtime environment or vehicle hardware is required.

### Tools Used

-   **Microsoft Word / LibreOffice Writer** --- Report preparation
-   **Python + Matplotlib** --- Architecture diagram
-   **Draw.io / diagrams.net** --- Diagram modeling/redrawing
-   **AOSP Documentation** --- Android Automotive component and
    interface references
-   **Android for Cars Documentation** --- Media, navigation and
    projection concepts

### Environment

A normal laptop or desktop with a web browser is sufficient.

No Android Studio, Android emulator, Raspberry Pi, or real vehicle
hardware is required for the conceptual version.

------------------------------------------------------------------------

## 🧪 Testing & Verification

Since this project is conceptual, testing was performed through **design
verification rather than code execution**.

### Requirement Coverage

  Requirement              Result
  ------------------------ --------
  Media integration        ✅ Met
  Navigation integration   ✅ Met
  Projection integration   ✅ Met
  Interface annotations    ✅ Met
  Data-flow annotations    ✅ Met

### Consistency Checks

The architecture was also checked to ensure:

-   Every diagram interface is represented in the interface table.
-   No application directly connects to a HAL or hardware.
-   Every numbered data-flow marker maps to a documented step.
-   Each flow can be traced from source to destination.
-   Component and HAL names are consistent with AOSP terminology.

All documented consistency checks passed.

------------------------------------------------------------------------

## 📊 Project Results

The final design contains:

-   **6 architectural layers**
-   **13 component interfaces**
-   **3 end-to-end data flows**
-   **12 numbered data-flow steps**
-   **1 combined navigation-and-music scenario**
-   **5 requirement checks**
-   **5 consistency checks**

The design demonstrates how media, navigation, smartphone projection,
audio management, location services, and vehicle data can work together
within an Android Automotive-style architecture.

------------------------------------------------------------------------

## ⚠️ Scope & Limitations

This project is a **conceptual architecture study**.

It does not include:

-   Real vehicle hardware
-   Real CAN bus communication
-   Production IVI source code
-   Executed software
-   Real-time performance measurements
-   Android Automotive emulator testing
-   Actual Android Auto or Apple CarPlay implementation

Some real-world Android Automotive components are simplified or grouped
together to keep the architecture readable.

For example, the native audio server and audio policy are represented
within the simplified Audio HAL connection.

------------------------------------------------------------------------

## 🔐 Safety & Quality Considerations

The architecture considers several important automotive concerns:

### Driver Distraction

Navigation and projection interfaces should limit complex interactions
while the vehicle is moving.

### Security

Applications and connected smartphones should not receive unrestricted
access to vehicle networks.

### Latency

Touch interactions and audio should respond quickly. Safety-critical
functions such as warnings and rear-view camera functionality generally
require stricter timing than normal infotainment features.

### Maintainability

The layered design allows different components to be updated or tested
independently.

### Updates

Future OTA mechanisms can update applications and services without
directly modifying safety-critical ECUs.

------------------------------------------------------------------------

## 🚀 Future Scope

The project can be extended in several directions:

-   Implement an Android Automotive media or navigation application.
-   Test the system using an Android Automotive emulator or Raspberry Pi
    setup.
-   Simulate Media Service and Audio HAL message flows using Android
    logs.
-   Model additional IVI use cases using UML.
-   Add Bluetooth telephony.
-   Add rear-view camera functionality.
-   Add instrument cluster integration.
-   Add multi-zone audio.
-   Add OTA update architecture.
-   Add cybersecurity components.
-   Measure real-world latency and system startup time.

------------------------------------------------------------------------

## 📚 References

The project is based primarily on Android Automotive and Android for
Cars documentation:

1.  Android Open Source Project --- Android Automotive OS
2.  Android Open Source Project --- Vehicle HAL
3.  Android Open Source Project --- Audio in Android Automotive
4.  Android Developers --- Android for Cars
5.  Google --- Android Auto
6.  Apple --- CarPlay
7.  Module 6: IVI --- In-Vehicle Infotainment Systems course material

------------------------------------------------------------------------

## 👨‍💻 Project Information

**Project:** IVI --- In-Vehicle Infotainment Systems\
**Topic:** Conceptual Architecture of an IVI System\
**Focus:** Media, Navigation and Smartphone Projection\
**Reference Platform:** Android Automotive Architecture\
**Author:** Rohit Roy\
**Course:** Wipro Automotive Course\
**Department:** Electronics and Communication Engineering\
**Institution:** Institute of Engineering and Management, Newtown

------------------------------------------------------------------------

## 📌 Summary

This project presents a conceptual six-layer architecture for an
**In-Vehicle Infotainment system** integrating media playback,
navigation, and smartphone projection.

The architecture uses concepts such as **Binder IPC, AIDL, Car Service,
Vehicle HAL, GNSS HAL, Audio HAL, audio focus, CAN, USB and Wi-Fi** to
explain how different parts of an automotive infotainment system
communicate.

The model provides a foundation for understanding how modern IVI systems
separate applications, services, automotive logic, hardware abstraction,
and physical devices.
