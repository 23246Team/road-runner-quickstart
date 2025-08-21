# Road Runner Tuning Guide (Two Odometry Pods + Gyro)

This guide provides detailed, step-by-step instructions for tuning Road Runner specifically for robots using **two odometry pods** ("dead wheels") and a **gyro** (IMU), rather than drive encoders or three-pod setups. Follow each section carefully to ensure optimal path following and localization.

---

## **General Guidance**

- **Read thoroughly!** Steps often depend on previous steps being fully completed.
- **Only follow steps relevant to two-pod odometry + gyro**. Ignore steps for drive encoders, three odometry pods, or OTOS.
- **Retune after any major hardware change** (weight, encoders, motors, etc).
- **Familiarity with FTC code in Android Studio is assumed.** Review [Learn Java for FTC](https://gm0.org/en/latest/docs/software/tutorials/learn-java.html) if needed.

---

## **Getting Help**

- **FTC Discord**: Great for practical advice from others who’ve tuned Road Runner.
- **Update Road Runner**: Ensure you’re using the latest version. Check `TeamCode/build.gradle`:
  ```
  implementation "com.acmerobotics.roadrunner:ftc:LIBRARY_VERSION_HERE"
  ```
- **For support**: Supply tuning files, code links, library version, and logs.

---

## **Drive Classes**

### **Goal:** Get basic hardware configured

1. **Choose and configure your Drive class**:
    - Use `MecanumDrive` or `TankDrive` (as appropriate for your drivetrain).
    - **Odometry**: Change the localizer to `TwoDeadWheelLocalizer`.
    - Update motor/encoder names as needed for your hardware.

2. **Set up the Localizer**:
    ```java
    // In your drive class:
    localizer = new TwoDeadWheelLocalizer(hardwareMap, lazyImu.get(), PARAMS.inPerTick, pose);
    ```
    - By default, expects encoders named `"par"` (parallel/forward) and `"perp"` (perpendicular).
    - The IMU should be named `"imu"`.
    - Adjust names to match your configuration if needed.

3. **IMU Setup**:
    - Follow FTC docs to set proper logo and USB orientation for the IMU.

4. **Check motor/encoder direction**:
    - Positive power should move robot forward and encoders should count up.
    - Use the `DeadWheelDirectionDebugger` opmode to verify encoder directions.
    - Reverse encoders in code if necessary.
    - **Note:** Encoder direction is independent of the motor it’s attached to.

5. **Set `DRIVE_CLASS` in `TuningOpModes`** to your drive class.

6. **Delete TODO comments** in code as you complete steps.

---

## **FTC Dashboard**

- Connect to the Program & Manage network.
- Go to:
    - `http://192.168.43.1:8080/dash` (Control Hub)
    - `http://192.168.49.1:8080/dash` (RC phone)
- Dashboard is used for tuning variables and live graphing.

---

## **Empirical Tuning Steps**

### 1. **ForwardPushTest**

**Goal:** Determine `inPerTick` empirically

- Place robot with space ahead. Note the starting position.
- Run `ForwardPushTest`.
- Slowly push robot straight forward.
- **Record**:
    - "Ticks traveled" (from telemetry)
    - Actual distance traveled (in inches, as measured on the field tiles)
- **Set**:
    ```java
    PARAMS.inPerTick = actual_distance_in_inches / ticks_traveled;
    ```
- If ticks are near zero, check encoder directions!

---

### 2. **ForwardRampLogger**

**Goal:** Find `kS` and `kV` feedforward parameters

- Place robot with clear space.
- Run `ForwardRampLogger`. The robot will slowly accelerate forward.
- **Stop** before robot reaches the edge.
- Download data from the dashboard:
    - `http://192.168.43.1:8080/tuning/forward-ramp.html`
- Click "Latest" to view results.
- **Exclude outliers** on the plot using the "e" key or "exclude" button.
- **Copy** `kS` and `kV` values into your drive class.

---

### 3. **AngularRampLogger**

**Goal:** Find `trackWidthTicks`, `kS`, and `kV` for rotation

- Run `AngularRampLogger`. The robot will rotate in place at increasing speed.
- Go to:
    - `http://192.168.43.1:8080/tuning/dead-wheel-angular-ramp.html`
- Click "Latest".
- **Copy** previously determined `kS`, `kV` into the boxes and click update.
- Use the outlier-exclusion technique for the "Track Width Regression" plot.
- **Set** `trackWidthTicks` to the "track width" value.
- **Scroll** for additional plots and record odometry wheel positions as instructed.

---

### 4. **ManualFeedforwardTuner**

**Goal:** Fine-tune `kS`, `kV`, and add `kA`

- Run `ManualFeedforwardTuner` from the dashboard.
- Graph `vref` against `v0`.
- Start with a small `kA` (e.g., `0.0000001`) and increase by factors of ten until it visibly affects the plot.
- Adjust to bring the two lines together as closely as possible.

---

### 5. **ManualFeedbackTuner**

**Goal:** Tune feedback (PID) parameters

- Run `ManualFeedbackTuner`.
- Adjust the controller’s feedback gains:
    - **Mecanum**: Forward, lateral, and heading position gains.
    - **Tank**: Ramsete gains (bBar and zeta).
- **Tips:**
    - Tweak one parameter at a time.
    - Start small (e.g., 1.0) and increase as needed.
    - Introduce disturbances and observe corrections.

---

### 6. **SplineTest**

**Goal:** Validate overall tuning

- Run `SplineTest` to follow a smooth spline.
- Robot should closely follow the path without significant deviation.

---

## **Notes and Best Practices**

- **Use ports 0 and 3 on REV hubs** for parallel pod encoders (better accuracy at high speeds).
- **Make code changes for hardware names** as needed for your robot.
- **Delete TODOs in code** as you go.

---

## **Summary Checklist**

- [ ] Drive class and localizer set to TwoDeadWheelLocalizer + gyro
- [ ] All motor and encoder directions checked
- [ ] `inPerTick` set empirically
- [ ] `kS`, `kV` determined and set
- [ ] `trackWidthTicks` set
- [ ] Odometry wheel positions set
- [ ] Feedforward and feedback terms tuned using dashboard
- [ ] SplineTest passes

---

## **References**

- [gm0 Odometry Guide](https://gm0.org/en/latest/docs/software/tutorials/odometry.html)
- [FTC Docs](https://ftc-docs.firstinspires.org/)
- [Road Runner Wiki](https://acmerobotics.github.io/road-runner/)
- [FTC Dashboard Docs](https://acmerobotics.github.io/ftc-dashboard/)

---

**Good luck with your tuning!**

