For two years of my studies I volunteered over twenty hours a week in [Ascend NTNU](https://www.ascendntnu.no/), the student organization at NTNU that builds autonomous drones for international competitions. The first year I was a perception engineer on [Team 2024](https://www.ascendntnu.no/teams/team-2024); from the spring of that year, deputy chief engineer, a role I held through [Team 2025](https://www.ascendntnu.no/teams/team-2025), sharing responsibility for the whole technical side.

## A drone that finds small things

Team 2024 built for [SUAS](https://suas-competition.org/), a competition in Maryland where an autonomous drone flies a course, locates objects on the ground and delivers payloads to them. The perception group turns the drone's cameras, radar and other sensors into what it knows about its surroundings, and my corner of that was object detection.

The hard part is scale. The drone searches from altitude, so a target on the ground is a handful of pixels in a high-resolution frame, and resizing the frame down to what a detection model expects erases the target entirely. The fix is slicing: cut each frame into tiles, run detection on every tile, and merge the results. The whole pipeline worked this way, and my part was the training side: I trained our YOLO-based detection model on sliced data, through a custom dataloader I wrote in Python that sliced the dataset on demand. There is a standard library for this, SAHI, but it fit our model and configuration poorly, and writing the slicing ourselves was simpler than bending the library to it.

Training ran on Idun, [NTNU's HPC cluster](https://www.hpc.ntnu.no/), through a harness I built and still keep public as [yolo-train](https://github.com/ahallemberg/yolo-train): distributed training across GPUs, hyperparameter tuning with Ray, experiment tracking with Weights & Biases. The budget on the other end was set by the drone itself: inference ran sliced too, on the onboard NVIDIA Jetson Orin, and part of my job was establishing how many tiles the Orin could push through in real time, since that decided what the model was allowed to look at. Around it all sat ROS 2, Python and C++.

In June 2024 we flew the competition in Maryland, and it ended on a hardware lesson no amount of code review could have caught: the mission had the drone flying faster than we had ever flown it in testing, a USB-C connector shook loose mid-course, and the drone stopped halfway. We found and fixed it the same day, but in the Maryland heat the judges would not allow a second run. We had traveled expecting to win, so it stung.

## Deputy chief engineer

In April 2024, while Team 24 was still building for Maryland, I stepped up as deputy chief engineer, and held the role through all of Team 2025. Together with the chief engineer I was responsible for the four technical groups: perception, control, autonomy and hardware. The work is coordination as engineering: setting the year's technical goals, keeping timelines honest, and making sure that what 29 engineers build in four groups integrates into one drone rather than four projects.

It also meant building the team itself. We recruited 21 new engineers from a pool of more than 200 applicants, the yearly renewal that keeps an organization of students alive as every member eventually graduates out of it.

The role also comes with a seat on Ascend's board, and that part of it was organizational rather than technical: setting up the technical budget, handling purchasing for the four groups, and running the organization together with the leaders of the business and marketing sides.

## Huntsville

Team 2025 built for [IARC Mission 10](https://www.ascendntnu.no/blog/team25-iarc-mission-10): drones that scan a minefield, detect the mines, and guide a person safely across, with voice control as part of the interface. Getting there was a project of its own: I had the main responsibility for moving the team's drones and equipment to the USA, which means an ATA Carnet for temporary import, customs stamps collected while the border offices are open at both ends of the journey, and lithium batteries taped, counted and spread across the team's carry-on within each airline's limits. In August 2025 we flew the mission in Huntsville, Alabama, and the days there compressed a year of work into its sharpest form: field debugging by day, fixes until four in the morning, and hard choices about which drone to bet on.

On the final run the drone flew the whole course, avoided the obstacles, including a tree in the middle of the minefield, and landed safely on the far side, while our person crossed the minefield within the time limit. Ascend was the only team to complete the course with a successful landing.

Voice control was the simplest system on board, a built-in speech API on an Android phone, and it simply worked: the drones were started by voice throughout the runs. The mine detection itself did not work that day, an optical filter tuned to the wrong band against a weaker signal than expected. The crossing leaned on what we could see rather than on what the system reported: most of the mines turned out to be visible, and luck covered the rest. The mission changed after that year, grading coordinates of detected mines instead of sending a person across. No team solved it outright, and none was really expected to: an IARC mission typically stands for years before it falls, and this was Mission 10's first year. A competition grades the whole system, and the system is exactly as strong as the last thing you had time to test.

## What it taught me

The perception year taught me what machine learning looks like when it has to fly: the model is one component among many, it shares a small computer with all of them, and a benchmark score means little if the tile budget blows the frame rate. That year also got me my first industry job, at [Q-Free](https://pages.askhb.no/Q-Free), along the way.

The deputy year taught the opposite kind of lesson, about work I could not do with my own hands. My output stopped being code and became whether four groups' work still fit together, and the two habits that carried the role were simple: ask questions, and end every meeting with what has been done and what must be ready by the next working day. Doing that beside a full course load also taught me what twenty hours a week actually costs, and that some things are worth it.
