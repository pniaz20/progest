# Learning-based Adaptive Admittance Controller for Efficient and Safe pHRI in Contact-Rich Manufacturing Tasks

This repository holds some detailed figures and plots related to the machine-learning models
developed in this paper for subtask detection and motion estimation in pHRI tasks that
involve contact with stiff and rigid environments, such as collaborative drilling, polishing, cutting, etc.

## Abstract

This paper proposes an adaptive admittance con-
troller for improving efficiency and safety in physical human-
robot interaction (pHRI) tasks in small-batch manufacturing
that involve contact with stiff environments, such as drilling,
polishing, cutting, etc. We aim to minimize human effort and
task completion time while maximizing precision and stability
during the contact of the machine tool attached to the robot’s
end-effector with the workpiece. To this end, a two-layered
learning-based human intention recognition mechanism is pro-
posed, utilizing only the kinematic and kinetic data from the
robot and two force sensors. A “subtask detector” recognizes
the human intent by estimating which phase of the task is
being performed, e.g., Idle, Tool-Attachment, Driving, and Contact.
Simultaneously, a “motion estimator” continuously quantifies
intent more precisely during the Driving phase to predict when
the Contact phase will begin. The controller is adapted online
according to the subtask while also allowing early adaptation
before the sensitive Contact phase to maximize precision and
safety and prevent potential instabilities. Three sets of pHRI
experiments were performed with multiple subjects under various
conditions. Spring compression experiments were performed in
virtual environments to train the data-driven models and validate
the proposed adaptive system, and drilling experiments were
performed in the real world to test the proposed methods’
efficacy in real-life scenarios. Experimental results show subtask
classification accuracy of 84% and motion estimation R2 score of
0.96. Furthermore, 57% lower human effort was achieved during
the Driving phase as well as 53% lower oscillation amplitude at
Contact as a result of the proposed system.

The contents of the repository are explained below.

## Experiments A

This directory has some detailed figures about data visualization and comparative model performances after conducting
virtual spring compression experiments with a collaborative robot and many human subjects.
These experiments were conducted to gather a training dataset for training the ML models.

### Data Visualization

This directory holds some figures that visualize the data collected in these experiments.

<img src="Experiment A/Data Visualization/Subtask_Statistics.png" alt="Subtask_Statistics">

[Fig. 1](Subtask_Statistics) above shows the distribution of the subtasks in the data collected in Experiment A (offline testing), in the virtual environment. It also shows the distribution of the lengths of the subtasks in the training set.

<img src="Experiment A/Data Visualization/Cartesian_Trajectories.png" alt="Cartesian_Trajectories">

[Fig. 2](Cartesian_Trajectories) above shows all Cartesian trajectories in space in all trials of Experiment A. The negative dimensions have been mirrored, and the perpendicular axis to the workpiece has been normalized. As can be seen, the trajectories hardly follow a straight-line minimum-jerk profile. The first row shows the small targets, and the bottom row shows the large targets.

<img src="Experiment A/Data Visualization/Velocity_Profiles.png" alt="Velocity_Profiles">

[Fig. 3](Velcocity_Profiles) above shows the profiles of the magnitude of Cartesian velocity in all trials of Experiment A (offline training) in the *Driving* phase. Different perpendicular travel distances are denoted with distinct colors. As can be seen, peak velocity, ranges of velocity at and before *Contact*, and the differences among the perpendicular travel distances are so vast that it is not easy to use predefined thresholds of velocity to decide when the Contact phase is imminent, and damping should be adapted. This shows the need for better data-driven methods for motion estimation in our study.

<img src="Experiment A/Data Visualization/Many_Plots.png" alt="Many_Plots">

[Fig. 4](Many_Plots) above provides statistical information about how human behaviors change as a function of perpendicular travel distance, in both small targets and large targets. Every subject is represented with a distinct color in these plots. The first row shows the time length of the *Driving* phase. As it can be seen, some subjects take distinctly longer or shorter than some others to complete the task. Also, in large targets, the time length is always less (faster) than the smaller targets, because smaller targets require more time to aim. The second row shows the average velocities during the *Driving* phase. There appears to be a positive relationship between perpendicular travel distance and the average velocity in *Driving*. That being said, once again, some subjects are distinctly faster than others. Furthermore, in large targets, the movement is distinctly faster than in smaller targets, for the aforementioned aiming reason. The bottom row of this figure shows the normalized time during the *Driving* phase when the velocity magnitude reaches its peak. In other words, it shows the relative time position of the peak velocity moment in all trials. As can be seen, there is no visible trend among the subjects and travel distances as to where the peak velocity occurs. This means that knowledge of the acceleration phase of the movement is not sufficient on its own to decide how the motion will continue and when the trial will end.

All of the findings above show the necessity for a data-driven model for motion estimation and subtask detection.

### Training Results

This directory includes some figures for the comparative test-set performances of the models under various cross-validation scenarios.

<img src="Experiment A/Training Results/Subtask_Detection.png" alt="Subtask_Detection">

[Fig. 5](Subtask_Detection) above shows the subtask detection performance of the LSTM and 1D CNN mdoels used in this study. Every row is a k-fold cross-validation category, in every fold of which all trials belonging to a single value of said category are used for testing, and all others are used for training. Every column is a metric. Boxplots show means and quartiles in this figure, while horizontal brackets show statistically significant pair-wise comparisons with $\alpha=0.001$ following a Bonferoni-corrected Wilcoxon signed rank test.

Metrics include the accuracy and F1 scores of subtask classification (4 classes; *Idle*, *Tool-Attachment*, *Driving* and *Contact*). Furthermore, two additional metrics are introduced: Concurrency and Fluctuation.
Concurrency is the accuracy of the model within 0.5-second intervals before and after subtask transitions. It shows the punctuality of the subtask detector in switching from one subtask to the other. This is important because we do not want lead or lag (premature or late detection) in our subtask detection, because that would lead to premature or delayed adaptation of the admittance controller damping, which is not ideal in our scenario. Fluctuation denotes the mean rate of fluctuation (in Hz) of the prediction of the subtask classifier outside transition regions. During each subtask, we do not want frequent fluctuations in the output of our subtask detector because such fluctuations would also lead to fluctuations in the damping values, which can jeopardize stability.

<img src="Experiment A/Training Results/Trajectory_Progress_Estimation.png" alt="Trajectory_Progress_Estimation">

[Fig. 6](Trajectory_Progress_Estimation) above shows the comparative performance of the motion estimator in estimating the trajectory progress during the *Driving* phase of the data collected in Experiment A (offline training). Once again, every row is a k-fold CV category, and every column is a metric. Once again, boxplots show means and quartiles in this figure, while horizontal brackets show statistically significant pair-wise comparisons with $\alpha=0.001$ following a Bonferoni-corrected Wilcoxon signed rank test.

Metrics include RMSE and R2 scores as with any regression problem, in addition to 2 new metrics: Max. Thresh. and Mismatch. Max. Thresh. (Maximum Adaptation Threshold) is the highest adaptation threshold (trajectory progress value at which we want to adapt the damping before *Contact* occurs) the model reaches, meaning the highest adaptation threshold that can be chosen to lead to successful adaptation before *Contact*. Mismatch denotes the punctuality of the model, i.e., the relative lead or lag of the model in terms of normalized trajectory length. It is the difference in normalized trajectory length in the *Driving* phase between the desired adaptation point (assuming the maximum allowable adaptation threshold was selected) and the real adaptation point.

Multiple models and methods can be seen in this figure, which can be summarized as follows:

- MJ: Simple minimum-jerk profile.
- DTW: Dynamic Time Warping. It takes the trajectory traveled so far in the *Driving* phase, and matches it with a template extracted from all trials in the training set, to get the best-matching portion of the template to the recorded trajectory.
- GPR: Gaussian Process Regression. Its input is a fixed-length sliding window sequence of data concatenated back-to-back to make a single feature vector, and its output is the trajectory progress value.
- LSTM and CNN: Deep learning models, whose inputs are sliding window sequences of data, and whose outputs are the instantaneous trajectory progress value.

As can be seen from the figure, the CNN model overall has the best performance, including the best regression performance, and the highest Max. Thresh and the lowest mistiming, which is why it was chosen in this study.
