# Problem Statement

## Problem Domain: Hip-hop/K-pop Dance
Learning and choreographing hip-hop/K-pop inspired dances. Dance is both a creative and physical outlet for me. I enjoy learning routines, experimenting with choreography, and collaborating with others. This domain excites me because it combines artistic expression, discipline, and community, and I am particularly drawn to projects that make learning and practicing dance more efficient and enjoyable.

## Problem: Difficulty Comparing Personal Dance Practices to Original Choreography
When trying to learn a K-pop dance, people will often put their dance video next to the original choreography and scroll through it frame by frame to see any differences or ways to refine their dance. This is a tedious process and it is difficult to remember every adjustment a dancer may want to make.

## Stakeholders
- **Casual dancers**: Beginner dancers who want to improve their form and choreography as they’re starting to dance.  
- **Professional dancers**: More experienced dancers who may be stricter on refining their movements in detail.  
- **Choreographers**: Create clearer and more structured choreography videos that make comparisons easier for learners.  
- **Dance instructors**: Benefit from dancers being able to self-correct with potential solutions, reducing the amount of active correction needed during lessons.  
- **Dance show audiences**: Experience more polished and cohesive performances when dancers refine their routines more effectively.  

## Evidence
1. **Dancing on the inside: A qualitative study on online dance learning with teacher-AI cooperation**  
   [Link](https://link.springer.com/article/10.1007/s10639-023-11649-0)  
   A participant noted that after receiving a lower dance score, they re-recorded their practice, highlighting how clearer feedback encourages refinement and practice.

2. **Reddit dancer advice (r/Dance)**  
   [Link](https://www.reddit.com/r/Dance/comments/czq4gs/comment/ez4n8ax)  
   A dancer emphasized breaking down moves into feet, legs, shoulders, and head, showing how dancers already see careful comparison as essential for improvement.

3. **SyncUp survey results**  
   [Link](https://arxiv.org/pdf/2107.13847)  
   Dancers rated the feature of receiving detailed textual or visual corrective feedback 4.66/5, showing a strong demand for comparison tools that guide improvement.

## Comparables
1. **Deep Dance**  
   [Link](https://devpost.com/software/deepdance)  
   A hackathon app that overlays a dancer’s “skeleton” next to the original choreography and provides a score. Limitation: it still requires manual review to locate areas needing improvement.

2. **Dance Sync**  
   [Link](https://github.com/Mruchus/dance-sync-analysis)  
   Compares user videos with a model dancer by analyzing joint angles and sync with music, then highlights mismatched frames. Limitation: runs only on MacOS and lacks an accessible UI.

3. **Dance Better AI**  
   [Link](https://www.dancebetter.org)  
   Users upload reference and practice videos for analysis of rhythm, posture, and technique, receiving feedback within 1–3 days. Limitation: behind a $10 paywall, unclear example feedback, and slow turnaround time.  

# Application Pitch

## Name  
**MirrorMotion**

## Motivation  
Learning a new dance can be exciting but also frustrating. Dancers often struggle to compare their practice videos with the original choreography, making it hard to pinpoint what to improve. MirrorMotion helps dancers bridge this gap with easy-to-use, precise feedback tools.

## Key Features  

### 1. Side-by-Side Sync  
Upload your practice video and the original choreography, and MirrorMotion automatically aligns them to the music.  
- **Why it helps**: Eliminates the tedious task of scrubbing frame by frame.  
- **Impact**: Beginners get a smoother way to learn routines, while professionals can drill down on timing for advanced refinement.  

### 2. Move Breakdown  
Using pose detection, MirrorMotion highlights specific body parts—arms, legs, torso—that are out of alignment with the choreography.  
- **Why it helps**: Provides targeted feedback on details dancers might otherwise miss.  
- **Impact**: Instructors and choreographers spend less time correcting basics in class and more time focusing on artistry and group cohesion.  

### 3. Practice Progress Tracker  
MirrorMotion stores past practice sessions, letting dancers compare improvements over time.  
- **Why it helps**: Transforms vague memories of “feeling better” into concrete progress metrics.  
- **Impact**: Builds motivation for casual learners and helps professionals refine their craft with measurable results.  

# Concept Design

## Concept: User  
**Purpose**: let users securely manage and access their own videos  
**Principle**: after registering with a username and password, the user can log in to upload, view, and edit their videos  

**State**  
A set of **User** with:  
- a username String  
- a password String  
- a videoIDs Set of Strings  

**Actions**  
- `register(username: String, password: String): (userID: String)`  
  - Requires: username not already taken  
  - Effect: creates a new User with login credentials  

- `login(username: String, password: String): (sessionID: String)`  
  - Requires: username exists and password matches  
  - Effect: authenticates user and creates a session  

- `uploadVideo(userID: String, videoID: String)`  
  - Requires: user exists, user has a valid session, and the video exists  
  - Effect: adds videoID to that user’s videoIDs set  

- `getUserVideos(userID: String): (videoID: Set of Strings)`  
  - Requires: user exists and is logged into their own account  
  - Effect: returns the list of videoIDs linked to that user  

- `deleteVideo(userID: String, videoID: String)`  
  - Requires: user exists, is logged in, and owns the specified video  
  - Effect: removes the videoID from the user’s videoIDs set  

---

## Concept: UploadVideo  
**Purpose**: allow dancers and choreographers to upload and manage practice/reference videos  
**Principle**: after uploading a video, it can be retrieved for analysis, syncing, or feedback  

**State**  
A set of **Videos** with:  
- a videoID String  
- a videoType of type practice or reference  
- a videoFile File  
- a owner User  
- a feedback Set of Feedback  

**Actions**  
- `upload(videoFile, videoType: String): (videoID: String)`  
  - Requires: videoType is practice or reference  
  - Effect: creates a new video entry with videoFile and videoType, stores it, and returns the unique videoID  

- `retrieve(videoID: String): (videoType, videoFile, feedback)`  
  - Requires: video exists  
  - Effect: returns the stored videoType, videoFile, and associated feedback  

- `delete(videoID: String)`  
  - Requires: video exists  
  - Effect: removes the video and its metadata  

---

## Concept: PoseBreakdown  
**Purpose**: extract poses from videos and represent them as collections of parts and points, which can later be compared  
**Principle**: after a video is processed, poses for each frame are stored as structured data  

**State**  
A set of **PoseData** with:  
- a poseID String  
- a partData of Set of PartData  

A set of **PartData** with:  
- a part Enum (Arm, Leg, etc.)  
- a pointData Set of PointData  

A set of **PointData** with:  
- a orderID Number  
- a location Number  

**Actions**  
- `extractPoses(videoID: String): (poseIDs: Set of String)`  
  - Requires: video exists  
  - Effect: processes each frame of the video, creates PoseData for each, and stores/returns their IDs  

- `getPoseData(poseID: String): (PoseData)`  
  - Requires: pose exists  
  - Effect: returns stored PoseData for inspection  

---

## Concept: Feedback  
**Purpose**: highlight differences between practice video and reference choreography  
**Principle**: after a video is broken into different poses, we can generate feedback on different body parts  

**State**  
A set of **Feedback** with:  
- a feedbackID String  
- a referencePoseData PoseData  
- a practicePoseData PoseData  
- a feedback String  
- a accuracyValue Number  

**Actions**  
- `analyze(referencePoseData: PoseData, practicePoseData: PoseData): (feedbackID: String)`  
  - Requires: both PoseData exist  
  - Effect: compares practice PoseData to reference PoseData, creates and stores Feedback  

- `getFeedback(feedbackID: String): (feedback: String, accuracyValue: Number)`  
  - Requires: feedback exists  
  - Effect: returns mismatched body parts with suggestions and accuracy score  

---

## Concept: ProgressTracker  
**Purpose**: log practice sessions for reference videos and track user improvement over time  
**Principle**: after logging a session, users can view their practice history and associated feedback across different reference videos  

**State**  
A set of **ProgressLogs** with:  
- a sessionID String  
- a owner User  
- a referenceVideo Video  
- a practiceVideos of Set of Videos  

**Actions**  
- `logSession(userID: String, referenceVideoID: String, practiceVideoID: String): (sessionID: String)`  
  - Requires: user, reference video, and practice video exist  
  - Effect: if no ProgressLog exists for (user, referenceVideo), create a new ProgressLog and include the practice video; otherwise, add the practice video to the existing ProgressLog. Returns the sessionID  

- `addVideo(sessionID: String, practiceVideoID: String)`  
  - Requires: ProgressLog exists for sessionID, and practice video exists  
  - Effect: adds the practice video to the ProgressLog’s practiceVideo set  

- `viewProgress(userID: String): (logs: Set of ProgressLogs)`  
  - Requires: user exists  
  - Effect: returns all ProgressLogs belonging to the user  

- `viewProgressByReference(userID: String, referenceVideoID: String): (practiceVideos: Set of Videos)`  
  - Requires: user and reference video exist, logs exist for that combination  
  - Effect: returns all practiceVideos for the user associated with the given reference video  


# Essential Synchronizations

## Sync: extractPoses  
**When**: `Video.upload(videoID, videoType)`  
**Then**: `PoseBreakdown.extractPoses(videoID)`  

---

## Sync: generateFeedback  
**When**: `PoseBreakdown.extractPoses(videoID): (poseIDs)`

**Where**: `videoID` is a practice video

**Then**: `Feedback.analyze(referencePoseData, practicePoseData)`  

---

## Sync: logProgress  
**When**: ``Video.upload(videoID, videoType)``  
**Then**: `ProgressTracker.logSession(userID, referenceVideoID, practiceVideoID)`  

# UI Sketches

![UI Sketch 1](../assets/assn2%20p1.jpg)


![UI Sketch 2](../assets/assn2%20p2.jpg)

# User Journey

Let’s say we have a **beginner dancer** who wants to learn a new K-pop choreography but struggles to identify exactly where her movements differ from the reference video.  

She opens **MirrorMotion** on her laptop, which takes her to her **personal dashboard** showing her previous practice sessions and uploaded videos. She clicks the **“Upload Video”** tab, and the app navigates to a page where she can select the file of her practice attempt and label it as a **reference video** or a **practice video**. She selects **pracitce video** and selects the **reference video** she wants to compare it against. Once she clicks **"“Submit”"**, the app stores her video and associates it with her account. 

After uploading, the app automatically processes her video using the **PoseBreakdown** system. She is then directed to the **Review Feedback page**, where the app displays her practice video alongside the reference choreography. She can see **highlighted differences in body parts**, with suggestions on which moves to refine (see wireframe for the **Review Feedback page**).

She can also navigate to her **Progress Tracker page** to see all her practice attempts for this choreography, compare improvements over time, and plan her next practice.

She feels motivated because the app makes it easy to identify mistakes, track her growth, and practice more effectively, without needing in-person coaching.  
