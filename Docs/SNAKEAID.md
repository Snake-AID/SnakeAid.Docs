| ![](data:image/png;base64...) | **MINISTRY OF EDUCATION AND TRAINING** |
| --- | --- |

| **FPT UNIVERSITY** |
| --- |
| Capstone Project Document |
| SnakeAid - AI-Powered Platform for Snakebite First Aid and Rescue Support |

| **GPSP26SE92** | |
| --- | --- |
| **Group Members** | Đoàn Ngọc Trung - SE183494  Phan Anh Khoa - SE183495  Nguyễn Văn Duy Khiêm - SE180168  Nguyễn Phúc Nhân - SE184696  Nguyễn Mạnh Dưỡng - SE181515 |
| **Supervisor** | Đỗ Tấn Nhàn |
| **Ext Supervisor** |  |
| **Capstone Project code** | SP26SE001 |

- Ho Chi Minh, January 2026 -

**Table of Contents**

[**Acknowledgement 15**](#_heading=h.ijnoujcdtkfy)

[**Definition and Acronyms 15**](#_heading=h.4lzal79pvrm3)

[**I. Project Introduction 16**](#_heading=h.kugeob1afmyn)

[1. Overview 16](#_heading=h.84tub4hag28w)

[1.1 Project Information 16](#_heading=h.ar84vtfv0g4i)

[1.2 Project Team 16](#_heading=h.3qy65f783eb)

[2. Product Background 16](#_heading=h.18beyk4tpjqr)

[3. Existing Systems 16](#_heading=h.a78djh3j7kcl)

[3.1 iNaturalist 16](#_heading=h.n63cr2rdolch)

[3.2 Vietnam Snakes 17](#_heading=h.h6nnlzwzvaki)

[3.3 Google Maps 17](#_heading=h.8qbz5lt4zck6)

[4. Business Opportunity 17](#_heading=h.eticy7os5jb0)

[5. Software Product Vision 17](#_heading=h.svvvly7b58fc)

[6. Project Scope & Limitations 18](#_heading=h.52daulfqcmq8)

[6.1 Project Scope 18](#_heading=h.ewn0vsjgs7wr)

[6.2 Limitations & Exclusions 22](#_heading=h.hzn0mjhivpoa)

[**II. Project Management Plan 24**](#_heading=h.ntlh9x5wphba)

[1. Overview 24](#_heading=h.sq7kxttk1uvu)

[1.1 Scope & Estimation 24](#_heading=h.yyql62650cly)

[1.2 Project Objectives 26](#_heading=)

[1.3 Project Risks 26](#_heading=h.9tnka8r7y2wi)

[2. Management Approach 27](#_heading=h.tgtcqiditauq)

[2.1 Project Process 27](#_heading=h.51qqofrrg7qr)

[2.2 Quality Management 29](#_heading=h.89xylkyq3x9y)

[2.3 Training Plan 30](#_heading=)

[3. Project Deliverables 30](#_heading=h.xagc861mkik2)

[4. Responsibility Assignments 31](#_heading=h.29wbve36jsg1)

[5. Project Communications 32](#_heading=)

[6. Configuration Management 32](#_heading=h.5d5zuxdfkx5m)

[6.1 Document Management 32](#_heading=h.98suzg1ovakt)

[6.2 Source Code Management 32](#_heading=h.xibovpdklrjo)

[6.3 Tools & Infrastructures 33](#_heading=h.wr94c3cfc82)

[**III. Software Requirement Specification 34**](#_heading=h.nhfa6yaaqbjd)

[1. Product Overview 34](#_heading=h.titwkxem8oav)

[2. User Requirements 35](#_heading=h.18chmer0prd5)

[2.1 Actors 35](#_heading=h.i8vb8pw4y7we)

[2.2 Use cases 36](#_heading=h.9wpxngz8ymm)

[2.2.1 Diagram 36](#_heading=h.2fom1fm0pjk3)

[2.2.2 Description 36](#_heading=h.raibbsfgujii)

[3. Functional Requirements 41](#_heading=h.fa0qc1ygtith)

[3.1 System Functional Overview 41](#_heading=h.e8qfmx34m0au)

[3.1.1 Screens Flow 41](#_heading=h.6dr4igue63hb)

[3.1.1.1 Member Screen Flow 41](#_heading=h.z62hekdyzp1q)

[3.1.1.2 Rescuer Screen Flow 42](#_heading=h.1m3yosgrfr3g)

[3.1.1.3 Expert Screen Flow 42](#_heading=h.w60vb8l9z816)

[3.1.1.4 Admin and Operator Screen Flow 43](#_heading=h.a286wcf0b8x4)

[3.1.2 Screen Descriptions 43](#_heading=h.sfl4jturhgdr)

[3.1.2.1 Member Screen Descriptions 43](#_heading=h.hxqvjxvd4fck)

[3.1.2.2 Rescuer Screen Descriptions 47](#_heading=h.xkwzrergwswb)

[3.1.2.3 Expert Screen Descriptions 49](#_heading=h.yr6t728qkxnc)

[3.1.2.4 Admin and Operator Screen Descriptions 51](#_heading=h.a5yqyy1y4xm)

[3.1.3 Screen Authorization 52](#_heading=h.byg397up4xmm)

[3.1.4 Non-Screen Functions 57](#_heading=h.xlqcc5x1dmqn)

[3.1.5 Entity Relationship Diagram 58](#_heading=h.h4bf3umutkcg)

[3.2 Authentication Feature 61](#_heading=)

[3.2.1 Mobile Login 61](#_heading=h.532iq7od9mi7)

[3.2.2 Forgot Password 63](#_heading=h.hbfd1577g0uh)

[3.2.3 Member Registration 64](#_heading=h.w5ru59yhlocn)

[3.2.4 Expert Registration 66](#_heading=h.chxpgbak4qj7)

[3.2.5 Rescuer Registration 68](#_heading=h.2tb94bt35rcv)

[Abnormal cases: Identity collision triggers "Account Exists" warning. 69](#_heading=h.4j3tfsm9yesh)

[3.2.6 Web Login 69](#_heading=h.evnme8l6oztt)

[3.3 Member Portal Feature 71](#_heading=h.y7zuphnsz2j0)

[3.3.1 Member Workspace & Notifications 71](#_heading=h.4fkefljrqngk)

[3.3.2 Community Alerts 74](#_heading=h.3m4kz3opuunl)

[3.3.3 Emergency SOS & Snake Identification 78](#_heading=h.wdrgetl96tal)

[3.3.4 Clinical Assessment & First Aid 85](#_heading=h.6d0cun96gqqw)

[3.3.5 Emergency Tracking & Incident Billing 89](#_heading=h.v4tjvzsmgyun)

[3.3.6 Snake Catching Request & Initial Payment 95](#_heading=h.q261f1dd7wg2)

[3.3.7 Snake Catching Tracking & Final Payment 103](#_heading=h.vxr2f1ias6pt)

[3.3.8 Scheduled Expert Consultation 108](#_heading=h.xtnn67hjvph0)

[3.3.9 Instant Expert Consultation 117](#_heading=h.8pxayh1nxq9s)

[3.4.10 Video Consultation & Completion 123](#_heading=h.n6sfkd53zy0y)

[3.3.11 Snake Library 127](#_heading=h.kq4r2k7bfkwb)

[3.3.12 Blog List 131](#_heading=h.iuy4mn7p7jn2)

[3.3.13 Top-up 134](#_heading=h.mm4tlyia1es4)

[3.3.14 Withdraw 135](#_heading=h.jdcobdqrest5)

[3.3.15 History Transaction 140](#_heading=h.3i3j4dmu05x1)

[3.3.16 Profile 142](#_heading=h.x8azy4pl275f)

[3.3.17 Activity History 144](#_heading=h.ny06u8mqy472)

[3.4 Rescuer Portal Feature 147](#_heading=h.9ysfvnhsde7y)

[3.4.1 Rescuer Workspace & Notifications 147](#_heading=h.5ggeci5j5uf6)

[3.4.2 Emergency Response Workflow 149](#_heading=h.mghasmd8qmsu)

[3.4.3 Snake Catching Workflow 155](#_heading=h.3uwcq3j2nihl)

[3.4.4 Profile & Feedback 160](#_heading=h.ixgf4yp92uw1)

[3.4.5 Mission History 161](#_heading=h.doe16lec825o)

[3.4.6 Lessons 163](#_heading=h.1z27a6glheuz)

[3.4.7 Snake Library 166](#_heading=h.91tcckocer75)

[3.5 Expert Portal Feature 169](#_heading=h.g1l4sqf16mcc)

[3.5.1 Expert Home & Notification 169](#_heading=h.3zf8upyy89ww)

[3.5.2 Profile & Settings 170](#_heading=h.dhrau35qbjp7)

[3.5.3 Expert Feedback 174](#_heading=h.hnfd8ibtjp77)

[3.5.4 AI Review Management 175](#_heading=h.xwnxqdj3eegs)

[3.5.5 Expert Knowledge Access 176](#_heading=h.rdbyf5ue9oya)

[3.5.6 Expert Blog Management 178](#_heading=h.lz9pcui8k5jb)

[3.5.7 Consultation Management 180](#_heading=h.tbs8k5ocpvws)

[3.5.8 Global Emergency Consultation Handling 183](#_heading=h.z9kd9i5xjp5e)

[3.6 Management Portal Feature 185](#_heading=h.hx2ojlbta2vv)

[3.6.1 Dashboard Management 185](#_heading=h.pf4oyhiwweoq)

[3.6.2 Workshifts Management 186](#_heading=h.eqwazay0tjo2)

[3.6.3 Users Management 186](#_heading=h.wl5lns2fsyxv)

[3.6.4 Request Management 187](#_heading=h.ydo7fn1yklds)

[3.6.5 Snakes Management 189](#_heading=h.ngnj0mbml3k)

[3.6.6 Antivenoms Management 190](#_heading=h.3sk0i0g5olwn)

[3.6.7 Treatment Facilities Management 191](#_heading=h.yi8q96rp5514)

[3.6.8 Transactions & Withdrawals Management 192](#_heading=h.nrfcpg8ho0fy)

[3.6.9 Settings Management 193](#_heading=h.1q21gvqz7zzk)

[3.6.10 Report Media Management 194](#_heading=h.3bp0br5im5nh)

[3.6.11 Lessons Management 195](#_heading=h.qkdx8w2q98d)

[3.6.12 Blogs Management 196](#_heading=h.e1k9e06af3sj)

[3.7 Operator Portal Feature 197](#_heading=h.xwgtl4bjkokz)

[3.7.1 Operator Dashboard 197](#_heading=h.crlgbgib4fcs)

[4. Non-Functional Requirements 199](#_heading=h.kgegcpyvwnim)

[4.1 External Interfaces 199](#_heading=h.a05di7a0ibku)

[4.2 Quality Attributes 199](#_heading=h.g6g1gcmj1rpa)

[4.2.1 Usability 199](#_heading=h.s66su4sqion9)

[4.2.2 Performance 199](#_heading=h.tvnhc0ofeycn)

[4.2.3 Reliability & Availability 200](#_heading=h.rc45hw6amhfd)

[4.2.4 Security 200](#_heading=h.qj5trcvs9fv9)

[4.2.5 Scalability 200](#_heading=h.aqswgzeu2paz)

[4.2.6 Maintainability 200](#_heading=h.i1ddtg3u7x4n)

[5. Requirement Appendix 200](#_heading=h.p36otfcpiafd)

[5.1 Business Rules 200](#_heading=h.lovjk5d1m5ny)

[5.2 Common Requirements 204](#_heading=)

[5.3 Application Messages List 204](#_heading=)

[**IV. Software Design Description 210**](#_heading=h.wwrdn0dad7lv)

[1. System Design 210](#_heading=h.bw18ev4wu7w1)

[1.1 System Architecture 210](#_heading=h.t9bd1qn55hhx)

[1.2 Package Diagram 211](#_heading=h.qyj14tw9jygb)

[2. Database Design 213](#_heading=h.5k2x3lc1dhxk)

[3. Detailed Design 219](#_heading=h.f8uhuzk64pgw)

[3.1 Snakebite Incident Emergency 219](#_heading=h.sg2xauhovvav)

[3.1.1 Class Diagram 219](#_heading=)

[3.1.2 Sequence Diagram Create SnakeBite Incident 220](#_heading=h.8ogcuwahaxle)

[3.1.3 Sequence Diagram Verify Snakebite Incident 220](#_heading=)

[3.1.4 Sequence Diagram False Alarm Snakebite Incident 221](#_heading=h.6h835r1tctvy)

[3.1.5 Sequence Diagram Dispatch Rescue Request 221](#_heading=h.j6zf3aworwhy)

[3.1.6 Sequence Diagram Accepts Rescue Request 222](#_heading=h.8wzcs2jvlorz)

[3.1.7 Sequence Diagram Decline Rescue Request 222](#_heading=h.tznl5d5ztay)

[3.1.8 Sequence Diagram Mark Rescue Mission Start 223](#_heading=h.ybdmwdf8zndl)

[3.1.9 Sequence Diagram Mark Rescue Mission Arrived 223](#_heading=h.lm02bwpazjb1)

[3.1.10 Sequence Diagram Mark Rescue Mission Complete 224](#_heading=h.8icik4ii6m2k)

[3.1.11 Sequence Diagram Cancel Snakebite Incident 224](#_heading=h.f0sxsm7v1wyr)

[3.1.12 Sequence Diagram Payment Snakebite Incident Wallet 225](#_heading=h.t9r6xz2c3e5c)

[3.1.13 Sequence Diagram Create Snakebite Incident PayOS Payment Link 225](#_heading=h.yf9uuy99bi6v)

[3.1.14 Sequence Diagram Payment Snakebite Incident via PayOS 226](#_heading=h.1vq8sd6b7ue2)

[3.2 Snake Capture 227](#_heading=h.jaevoni7hccz)

[3.2.1 Class Diagram 227](#_heading=h.7p7vg0b2chjd)

[3.2.2 Sequence Diagram Create new Snake Catching Request 228](#_heading=h.d8ihxthaqx5a)

[3.2.3 Sequence Diagram Confirm Snake Catching Request 228](#_heading=h.z6gs161txrb5)

[3.2.4 Sequence Diagram Assign Snake Catching Request 229](#_heading=h.32xfh0u0yszh)

[3.2.5 Sequence Diagram Cancel Snake Catching Request 229](#_heading=h.ge3pnsydu7ig)

[3.2.6 Sequence Diagram View List Snake Catching Request 230](#_heading=h.232s3w9reew7)

[3.2.7 Sequence Diagram Start Snake Catching Mission 230](#_heading=h.545bx94fl9se)

[3.2.8 Sequence Diagram Mark as Arrived Snake Catching Mission 231](#_heading=h.levp4nedpvkw)

[3.2.9 Sequence Diagram Complete Snake Catching Mission 231](#_heading=h.at3xxkure0a1)

[3.3 Consultation 232](#_heading=h.cg5vfx2mmkgl)

[3.3.1 Class Diagram 233](#_heading=h.sr7726hsfca9)

[3.3.2 Sequence Diagram View List Experts and Presence 233](#_heading=h.n2q3gwwvo750)

[3.3.3 Sequence Diagram Create and Pay Scheduled Booking 233](#_heading=h.gvmc0ord4sps)

[3.3.4 Sequence Diagram Create, Pay, and Notify Emergency Consultation Request 234](#_heading=h.yn81qo2fs3id)

[3.3.5 Sequence Diagram Expert Accept/Reject Emergency Request 235](#_heading=h.3drv81fxquva)

[3.3.6 Sequence Diagram Join Consultation Room and In-Room Interaction 235](#_heading=h.402txxvzawdp)

[3.3.7 Sequence Diagram End Consultation and Settlement 236](#_heading=h.x6tn5v71px0x)

[3.3.8 Sequence Diagram Create Consultation Review 236](#_heading=h.ue9eg9q7acbu)

[**V. Software Testing Documentation 237**](#_heading=)

[1. Scope of Testing 237](#_heading=h.9ubyg4copqzk)

[2. Test Strategy 238](#_heading=h.4y938j4vzpmu)

[2.1 Testing Types 238](#_heading=h.yinjubywsfm0)

[2.2 Test Levels 238](#_heading=h.cforhaaphodq)

[2.3 Supporting Tools 238](#_heading=h.tmpi0dajt0qi)

[3. Test Plan 239](#_heading=h.pqemzbvsx902)

[3.1 Human Resources 239](#_heading=h.liehl9g875yh)

[3.2 Test Environment 239](#_heading=h.ae6mzw8opjm8)

[3.3 Test Milestones 239](#_heading=h.v75dpwnsk39)

[4. Test Cases 240](#_heading=h.bfzi3dumlrfr)

[4.1 Unit Test Cases 240](#_heading=h.pgmnovq9rgc9)

[4.2 Other Test Cases (IT,ST,AT) 241](#_heading=h.1liye9we4377)

[5. Test Reports 242](#_heading=h.wlik4mla77fp)

[5.1 Unit Test Report 242](#_heading=h.eggcy87tpi0l)

[5.2 Other Test Report (IT,ST,AT) 242](#_heading=h.yqct60d1tuji)

[**VI. Release Package & User Guides 244**](#_heading=h.jmiqe544qhhz)

[1. Deliverable Package 244](#_heading=h.o1ptvmggte1d)

[2. Installation Guides 244](#_heading=h.stvwf59cta61)

[2.1 System Requirements 244](#_heading=h.ossoya9sxwp9)

[2.2 Installation Instruction 244](#_heading=h.eu99oibmbol)

[2.2.1 Frontend 244](#_heading=h.efxyf54e5wkh)

[2.2.1.1 Frontend Mobile Application 244](#_heading=h.gnxdnlbqbj3e)

[2.2.1.2 Frontend Web Application 247](#_heading=h.hxysj13asjpo)

[2.2.2 Backend 249](#_heading=h.lhoqgaqcqng7)

[2.2.3 Ai Inference Server 250](#_heading=h.sknc6prukv15)

[3. User Manual 252](#_heading=h.c7nld5ttcua8)

[3.1 Overview 252](#_heading=h.xvzfx73qcy7c)

[3.2 SOS Snake Bite 255](#_heading=h.sgjitvbm7qrv)

[3.3 Snake Catching 293](#_heading=h.6dh3w0xhc9np)

[3.4 Scheduled Consultation 327](#_heading=h.91896l4ae8l2)

[3.5 Instant Consultation 339](#_heading=h.nwenq2qwzu14)

[3.6 System Administration 353](#_heading=h.19cauimlgavb)

**List Of Tables And Figures**

[Table 1.1 Definition and Acronyms 15](#_heading=h.gykxe9ye90vz)

[Table 1.1.2.1 Project’s member list 16](#_heading=h.bg88mp2armhb)

[Table 2.1.1.1 Scope & Estimation 26](#_heading=h.gyagsxwbpcxj)

[Table 2.1.2.1 Project Objectives 26](#_heading=h.gq8l8r78nhyo)

[Table 2.1.3.1 Project Risks 27](#_heading=h.6v1uk66z41yo)

[Figure 2.2.2.1 Agile Scrum Cycle 28](#_heading=h.njiycxssrl3k)

[Table 2.2.3.1 Training Plan 30](#_heading=h.9vvkwmwvkci4)

[Table 2.4.1 Responsibility Assignments 31](#_heading=h.iyjonwcqzk93)

[Table 2.5.1 Project Communication 32](#_heading=h.do1deuqzufby)

[Table 2.6.3.1 Tools & Infrastructures 33](#_heading=h.x5tal4967asp)

[Table 3.2.2.1 Actors 35](#_heading=h.6r5g00dfiogp)

[Figure 3.2.2.1.1 Use Case Diagram 36](#_heading=h.39f7b1cu953g)

[Table 3.2.2.2.1 Use Case Description 41](#_heading=h.3hvon5phovgn)

[Figure 3.3.1.1.1.1 Member Screen Flow 41](#_heading=h.vgsxftmb2z0o)

[Figure 3.3.1.1.1.2 Rescuer Screen Flow 42](#_heading=h.d1mc5j983y4f)

[Figure 3.3.1.1.1.3 Expert Screen Flow 42](#_heading=h.kttuvy78rd02)

[Figure 3.3.1.1.4 Admin and Operator Screen Flow 43](#_heading=h.n598ez37yyjw)

[Table 3.3.1.2.1 Member Screen Description 47](#_heading=h.oeyed4fx66va)

[Table 3.3.1.2.2 Rescuer Screen Description 49](#_heading=h.3ocv268heo30)

[Table 3.3.1.2.3 Expert Screen Description 51](#_heading=h.gvl31jm9zths)

[Table 3.3.1.2.4 Admin and Operator Screen Description 52](#_heading=h.lhdba0y7iim8)

[Table 3.3.1.3.1 Screen Authorization 57](#_heading=h.8vjv4c19d14r)

[Table 3.3.1.4.1 Non-Screen Functions 57](#_heading=h.og93jlwclf34)

[Figure 3.3.1.5.1 Entity Relationship Diagram 58](#_heading=h.jak5r6u4n0s)

[Table 3.3.1.5.2 Entities Description 60](#_heading=h.hfrojrlked5p)

[Figure 3.3.2.1.1 Splash Screen 61](#_heading=h.m4adqey7v61c)

[Figure 3.3.2.1.2 Expert Login Screen 62](#_heading=h.gsunrlc52gh2)

[Figure 3.3.2.2.1 Forgot Password Screen 64](#_heading=h.vj620ghin1fi)

[Figure 3.3.2.3.1 Member Registration Screen 64](#_heading=h.6m73gf2frgxd)

[Figure 3.3.2.3.2 Registration Success Screen 65](#_heading=h.5xfk3sfo9dbl)

[Figure 3.3.2.4.1 Expert Registration Screen 66](#_heading=h.q536p0uc43t6)

[Figure 3.3.2.4.2 Expert Registration Success Screen 67](#_heading=h.1ieg69gid2ls)

[Figure 3.3.2.4.3 Expert Credentials Screen 68](#_heading=h.la0hywx8722w)

[Figure 3.3.2.5.1 Rescuer Registration Modal 69](#_heading=h.v9mdw62mz9wb)

[Figure 3.3.2.6.1 Web Login Screen 70](#_heading=h.cao61yngzyff)

[Figure 3.3.2.6.2 Admin Login Form Screen 70](#_heading=h.kwsl87a5m33p)

[Figure 3.3.3.1.1 Member Home Screen 72](#_heading=h.8wmtwjma17ps)

[Figure 3.3.3.1.2 Notification Tab Screen 73](#_heading=h.eegf4h9mx5t0)

[Figure 3.3.3.2.1 Community Alert Screen 74](#_heading=h.4667kc86wweu)

[Figure 3.3.3.2.2 Upload Community Report Screen 75](#_heading=h.hvf36jt1csfk)

[Figure 3.3.3.2.3 History Community Screen 76](#_heading=h.4662cjp6sqoy)

[Figure 3.3.3.2.4 Edit Community Report Screen 77](#_heading=h.bwaafrtetvb)

[Figure 3.3.3.3.1 Emergency Action Screen 79](#_heading=h.1pdv67nntob4)

[Figure 3.3.3.3.2 Emergency Tracking Screen 80](#_heading=h.vk691mo36va3)

[Figure 3.3.3.3.3 Snake Identification Screen 81](#_heading=h.g08wyexdfefh)

[Figure 3.3.3.3.4 Snake Confirmation Screen 82](#_heading=h.4b6gpndbeequ)

[Figure 3.3.3.3.5 Snake Selection by Location Screen 83](#_heading=h.20fd63khg0ny)

[Figure 3.3.3.3.6 Confirm Snake Selection by Location Modal 84](#_heading=h.c9rso4pwy2up)

[Figure 3.3.3.4.1 First Aid Steps Screen 86](#_heading=h.aefv8kqjzx8p)

[Figure 3.3.3.4.2 Symptom Report Screen 87](#_heading=h.syj4peb5azri)

[Figure 3.3.3.4.3 Severity Assessment Screen 88](#_heading=h.8sf9k8m8pua1)

[Figure 3.3.3.5.1 Emergency Tracking Screen - Status Mission “En-route” 90](#_heading=h.n0ezmjtgwlq)

[Figure 3.3.3.5.2 Rescuer Arrived Screen 91](#_heading=h.mkdkaspqqqqd)

[Figure 3.3.3.5.3 Member Incident Finished Screen 92](#_heading=h.p1upr5p2dno1)

[Figure 3.3.3.5.4 Member Selection Payment Modal 93](#_heading=h.8eb40z44t3oc)

[Figure 3.3.3.5.5 Member Payment Successful Modal 94](#_heading=h.kabgitkcj4yt)

[Figure 3.3.3.6.1 Snake Quantity Selection Screen 96](#_heading=h.ppraqsw30m9m)

[Figure 3.3.3.6.2 Snake Report Detail Screen 97](#_heading=h.1s17a1gxu1bv)

[Figure 3.3.3.6.3 Snake Report Detail Screen (AI result) 98](#_heading=h.i55858n6o62m)

[Figure 3.3.3.6.4 Snake Catching Success Screen 99](#_heading=h.m91bcf2ndkv)

[Figure 3.3.3.6.5 Member Selection Payment Modal (Travel Fee) 100](#_heading=h.wofo8r41g05o)

[Figure 3.3.3.6.6 Member Payment Successful Modal (Travel Fee) 101](#_heading=h.z3talo4zj629)

[Figure 3.3.3.6.7 Snake Catching Detail Screen 102](#_heading=h.u28tbc8qhrkl)

[Figure 3.3.3.7.1 Snake Catching Detail Screen (when status “En-Route”) 104](#_heading=h.3voll1hbg7a)

[Figure 3.3.3.7.2 Snake Catching Detail Screen (when status “Arrived”) 105](#_heading=h.3wpdf1een5an)

[Figure 3.3.3.7.3 Member Selection Payment (Catching Payment) 106](#_heading=h.7wp7tnhnc3lr)

[Figure 3.3.3.7.4 Snake Catching Detail Screen (when status “Completed”) 107](#_heading=h.csfy2373otzh)

[Figure 3.3.3.8.1 Consultation Home Screen 109](#_heading=h.p09et4fjpd9c)

[Figure 3.3.3.8.2 Expert List Screen 110](#_heading=h.qcz0qrwuhh4b)

[Figure 3.3.3.8.3 Expert Detail Screen 111](#_heading=h.fa45wymxu2ve)

[Figure 3.3.3.8.4 Service Selection Screen 112](#_heading=h.kdiwlmxbsdko)

[Figure 3.3.3.8.5 Scheduled Consultation Screen 113](#_heading=h.s49598u53w82)

[Figure 3.3.3.8.6 Consultation Documents Screen 114](#_heading=h.ns6bk5vixbgg)

[Figure 3.3.3.8.7 Payment Confirmation Screen 115](#_heading=h.l7zzvkqwihdv)

[Figure 3.3.3.8.8 Consultation Home Screen (when it have the booking) 116](#_heading=h.ctxbyxdl1ak)

[Figure 3.3.3.9.1 Expert List Screen (Have Expert Online) 118](#_heading=h.6shpgabx0ugr)

[Figure 3.3.3.9.2 Expert Detail Screen (Expert Online) 119](#_heading=h.w101m4fip25x)

[Figure 3.3.3.9.3 Service Selection Screen (Expert Online) 120](#_heading=h.qde1k2w8a59c)

[Figure 3.3.3.9.4 Payment Confirmation Screen 121](#_heading=h.rl8704w6buh)

[Figure 3.3.3.9.5 Emergency Request Waiting Modal 122](#_heading=h.3oj8nfjps0nb)

[Figure 3.3.3.10.1 Video Waiting Room Screen 123](#_heading=h.xgj5yixp27oi)

[Figure 3.3.3.10.2 Video Consultation Screen 124](#_heading=h.vens5p1b0424)

[Figure 3.3.3.10.3 Box Chat Modal 125](#_heading=h.1vo8duo9mfw3)

[Figure 3.3.3.10.4 Consultation Complete Screen 126](#_heading=h.kvj79wknlvtv)

[Figure 3.3.3.11.1 Snake Library Screen 128](#_heading=h.flo7hmnn1d6r)

[Figure 3.3.3.11.2 Snake Detail Screen 129](#_heading=h.dqli7no9xglg)

[Figure 3.3.3.11.3 Snake First Aid Guide Screen 130](#_heading=h.m7jcjqes7cuo)

[Figure 3.3.3.12.1 Blog List Screen 131](#_heading=h.n8bmjumw9wfw)

[Figure 3.3.3.12.2 Blog Detail Screen (with unlike) 132](#_heading=h.jeokjnrhx69j)

[Figure 3.3.3.12.3 Blog Detail Screen (with like) 133](#_heading=h.42w8djt0qa32)

[Figure 3.3.3.13.1 Top-up Screen 134](#_heading=h.yrqknhypxi5x)

[Figure 3.3.3.14.1 Withdrawal Screen 136](#_heading=h.w3iqnpq7l4bx)

[Figure 3.3.3.14.2 Withdrawal Request Modal 137](#_heading=h.l2l4wkuwvwf8)

[Figure 3.3.3.14.3 Withdrawal Detail Screen (with status Pending) 138](#_heading=h.j2mh13sbkcu4)

[Figure 3.3.3.14.4 Withdrawal Detail Screen (with status Approved) 139](#_heading=h.actbtlf98rpo)

[Figure 3.3.3.15.1 History Transaction Screen 140](#_heading=h.ld1x55nlns7z)

[Figure 3.3.3.15.2 Transaction Detail Screen 141](#_heading=h.t2jl8ue0s9zy)

[Figure 3.3.3.16.1 Profile Tab Screen 142](#_heading=h.gygkpmlcoz1y)

[Figure 3.3.3.16.2 Edit Profile Screen 143](#_heading=h.4qv4p9c6du1c)

[Figure 3.3.3.17.1 Activity Tab Screen 144](#_heading=h.hdtace4ofh1a)

[Figure 3.3.3.17.2 Activity History Screen 145](#_heading=h.vowmhwnkphay)

[Figure 3.3.3.17.3 Activity Detail Screen 146](#_heading=h.osx0vb39rzlk)

[Figure 3.3.4.1.1 Rescuer Home And Notification Screen 147](#_heading=h.7dsyag8ebjzs)

[Figure 3.3.4.1.2 Work Schedule Tab Screen 148](#_heading=h.yxy7s0hyekl5)

[Figure 3.3.4.2.1 Mission Detail - SOS Screen 149](#_heading=h.r52b5m7wtdxr)

[Figure 3.3.4.2.2 Navigation Map Screen 150](#_heading=h.okwjx7lkjm8y)

[Figure 3.3.4.2.3 On-scene Support Screen 151](#_heading=h.lbc34cfbrb0n)

[Figure 3.3.4.2.4 Find Hospital Screen 152](#_heading=h.n0nylsm5h4f2)

[Figure 3.3.4.2.5 Mission Completion Screen 153](#_heading=h.al0mlisri8cg)

[Figure 3.3.4.2.6 Mission Success - Emergency Screen 154](#_heading=h.wz3tgu6t2r24)

[Figure 3.3.4.3.1 Request Catching Detail 156](#_heading=h.6kc24nk3pmac)

[Figure 3.3.4.3.2 En Route Screen 157](#_heading=h.3sswqij8gmgc)

[Figure 3.3.4.3.3 Tracking Screen 158](#_heading=h.uh5i1z44o4ez)

[Figure 3.3.4.3.4 Result Confirmation & Mission Success - Snake Catching Screen 159](#_heading=h.6winhm3c3vqw)

[Figure 3.3.4.4.1 Profile Tab Screen 160](#_heading=h.ol01cq5pqos5)

[Figure 3.3.4.4.2 Feedback Screen 161](#_heading=h.40beggdhizsv)

[Figure 3.3.4.5.1 Mission History Screen 162](#_heading=h.vaxjmgnsnptm)

[Figure 3.3.4.5.2 History Detail Screen 163](#_heading=h.puz0kf71j4pd)

[Figure 3.3.4.6.1 Lessons Screen 164](#_heading=h.e5o0athf6yri)

[Figure 3.3.4.6.2 Lesson Detail Screen 165](#_heading=h.a94i11wdizsa)

[Figure 3.3.4.7.1 Snake Library Screen (Rescuer) 166](#_heading=h.rszj4najwtrt)

[Figure 3.3.4.7.2 Snake Detail Screen (Rescuer) 167](#_heading=h.efnacxmbwqyj)

[Figure 3.3.4.7.3 First Aid Guide (Rescuer) 168](#_heading=h.w1ybx4ejkize)

[Figure 3.3.5.1.1 Home Screen 169](#_heading=h.esw5h3hix7le)

[Figure 3.3.5.2.1 Profile Tab Screen 171](#_heading=h.exuhnklk71fe)

[Figure 3.3.5.2.2 Withdraw Screen 172](#_heading=h.xf2spb94pus1)

[Figure 3.3.5.2.3 Working Hours Screen 173](#_heading=h.3vhzxypvv6i3)

[Figure 3.3.5.3.1 Feedback Screen 174](#_heading=h.7zvnrap6laz4)

[Figure 3.3.5.4.1 AI Review Screen 175](#_heading=h.lti537q39zpx)

[Figure 3.3.5.5.1 Snake Library and Snake Detail Screen 177](#_heading=h.anaseohx77c0)

[Figure 3.3.5.5.2 Snake First Aid Screen 178](#_heading=h.qmq5iqzddcmx)

[Figure 3.3.5.6.1 Blog List Screen 179](#_heading=h.bhfvsaggtkbn)

[Figure 3.3.5.6.2 Expert Blog Form Screen 180](#_heading=h.9pbwroomfa0c)

[Figure 3.3.5.7.1 Video Waiting and Video Consultation Screen 181](#_heading=h.29tqrrtxnhel)

[Figure 3.3.5.7.2 Chat and Look Up Snake Species Screen 182](#_heading=h.156s82dk4ggj)

[Figure 3.3.5.7.3 Consultation History Tab 183](#_heading=h.ug6ls1l6jgbf)

[Figure 3.3.5.8.1 Expert Global Emergency Popup Listener Screen 184](#_heading=h.j8odvjq6l4zq)

[Figure 3.3.6.1.1 Admin Dashboard 185](#_heading=h.bsnzuar0ahlu)

[Figure 3.3.6.2.1 Workshifts Management Screen 186](#_heading=h.6u82y7s96vtx)

[Figure 3.3.6.3.1 Users Management Screen 187](#_heading=h.tgqgmexi67my)

[Figure 3.3.6.4.1 Incidents Management Screen 188](#_heading=h.rs58wdf8ypwv)

[Figure 3.3.6.4.2 Snake Catching Management Screen 188](#_heading=h.idoau1c2z7d2)

[Figure 3.3.6.4.3 Consultations Management Screen 189](#_heading=h.d8bvml1okje4)

[Figure 3.3.6.5.1 Snakes Management Screen 190](#_heading=h.ple3xwxkqewh)

[Figure 3.3.6.6.1 Antivenoms Management 191](#_heading=h.addrc6bxix31)

[Figure 3.3.6.7.1 Treatment Facilities Management Screen 192](#_heading=h.qkheeo9z8whq)

[Figure 3.3.6.8.1 Transactions Screen 193](#_heading=h.hce1re75tmtt)

[Figure 3.3.6.8.2 Withdrawals Screen 193](#_heading=h.16mrbqknkfbe)

[Figure 3.3.6.9.1 Settings Management Screen 194](#_heading=h.i5fs00f8sswl)

[Figure 3.3.6.10.1 Report Media Management Screen 195](#_heading=h.ewfaxncu0pm9)

[Figure 3.3.6.11.1 Lessons Management Screen 196](#_heading=h.x33ybu5y0u32)

[Figure 3.3.6.12.1 Blogs Management Screen 197](#_heading=h.xb8dwffemm0d)

[Figure 3.3.7.1.1 Operator Dashboard 198](#_heading=h.qt5y5ye01o5h)

[Figure 3.3.7.1.2 Request Detail Modal 198](#_heading=h.xacsaiunkqvb)

[Table 3.5.1.1 Business Rules 204](#_heading=h.aocdcom0tf9z)

[Table 3.5.3.1 Application Messages List 209](#_heading=h.b13zw0j1rmbi)

[Figure 4.1.1.1 System Architecture Diagram 210](#_heading=h.k8ua3j1u1tgx)

[Figure 4.1.1.2 Package Diagram 211](#_heading=h.o17srr4gjrqw)

[Table 4.1.1.3 Package Description 212](#_heading=h.4r2hhze3uizg)

[Figure 4.2.1 Physical Database 213](#_heading=h.n8op8rwt4dm8)

[Figure 4.2.2 Database Design Description 219](#_heading=h.510ai3dde44e)

[Figure 4.3.1.1.1 Snakebite Incident Class Diagram 219](#_heading=h.pzayemd1h0xb)

[Figure 4.3.1.2.1 Create Snakebite Incident Sequence Diagram 220](#_heading=h.ricx0kftxoxb)

[Figure 4.3.1.3.1 Verify Snakebite Incident Sequence Diagram 220](#_heading=h.asawrx5gzhi4)

[Figure 4.3.1.4.1 False Alarm Snakebite Incident Sequence Diagram 221](#_heading=h.8pc22q9q5r4y)

[Figure 4.3.1.5.1 Dispatch Rescue Request Sequence Diagram 221](#_heading=h.hj0gmmrf7sff)

[Figure 4.3.1.6.1 Accept Rescue Request Sequence Diagram 222](#_heading=h.lhpfpgekiw6e)

[Figure 4.3.1.7.1 Decline Rescue Request Sequence Diagram 222](#_heading=h.v0lxiw2wh8b2)

[Figure 4.3.1.8.1 Start Rescue Mission Sequence Diagram 223](#_heading=h.v2yrfbbqkma0)

[Figure 4.3.1.9.1 Mark Rescue Mission Arrived Sequence Diagram 223](#_heading=h.dqqxooq1yrxe)

[Figure 4.3.1.10.1 Complete Rescue Mission Sequence Diagram 224](#_heading=h.s5i9v35bnm1y)

[Figure 4.3.1.11.1 Complete Rescue Mission Sequence Diagram 224](#_heading=h.lfrt8yakevv)

[Figure 4.3.1.12.1 Payment Snakebite Incident by Wallet Sequence Diagram 225](#_heading=h.wslevoc3v1zg)

[Figure 4.3.1.13.1 Create Snakebite Incident PayOS Payment Link Sequence Diagram 225](#_heading=h.ti3gqybjx7s1)

[Figure 4.3.1.14.1 Payment Snakebite Incident via PayOS Sequence Diagram 226](#_heading=h.2z6u5hnrbu5m)

[Figure 4.3.2.1.1 Snake Capture Class Diagram 227](#_heading=h.scqth0t6wmmd)

[Figure 4.3.2.2.1 Create Snake Catching Request Sequence Diagram 228](#_heading=h.m4du935jjehu)

[Figure 4.3.2.3.1 Confirm Snake Catching Request Sequence Diagram 228](#_heading=h.7iqr58shj3ag)

[Figure 4.3.2.4.1 Assign Snake Catching Request Sequence Diagram 229](#_heading=h.bcr83fsrbo8l)

[Figure 4.3.2.5.1 Cancel Snake Catching Request Sequence Diagram 229](#_heading=h.o1evxbg0kdza)

[Figure 4.3.2.6.1 View List Catching Request Sequence Diagram 230](#_heading=h.94y57dbc173u)

[Figure 4.3.2.7.1 Start Snake Catching Mission Sequence Diagram 230](#_heading=h.pdo0alae8y72)

[Figure 4.3.2.8.1 Mark Arrived Snake Catching Mission Sequence Diagram 231](#_heading=h.sbj48xjmjr58)

[Figure 4.3.2.9.1 Complete Snake Catching Mission Sequence Diagram 231](#_heading=h.ljacwj8sdiu)

[Figure 4.3.3.1.1 Consultation Class Diagram 233](#_heading=h.obe48wxvyzjr)

[Figure 4.3.3.2.1 View List Expert and Presence Class Diagram 233](#_heading=h.7mt17lzgaud)

[Figure 4.3.3.3.1 Create and Pay Scheduled Booking Class Diagram 234](#_heading=h.m3v7aqh2qyfg)

[Figure 4.3.3.4.1 Create, Pay, Notify Emergency Consultation Request Class Diagram 234](#_heading=h.371vtebq6guj)

[Figure 4.3.3.5.1 Expert Accept or Reject Emergency Consultation Request Class Diagram 235](#_heading=h.a4ufzt7iv651)

[Figure 4.3.3.6.1 Join Consultation Room and In-Room Class Diagram 235](#_heading=h.hb9e41sanr73)

[Figure 4.3.3.7.1 End Consultation Class Diagram 236](#_heading=h.uudfroe2831q)

[Figure 4.3.3.8.1 Create Consultation Review Class Diagram 236](#_heading=h.azbdjykqgk3m)

[Table 5.1.1 Scope of Testing 237](#_heading=h.fpkaqo3cq3g2)

[Table 5.2.1.1 Testing Types 238](#_heading=h.ao5dy6mjhus8)

[Table 5.2.2.1 Test Levels 238](#_heading=h.mg8x8kcys4k5)

[Table 5.2.3.1 Supporting Tools 238](#_heading=h.gb5va9ke1xdb)

[Table 5.3.1.1 Human Resources 239](#_heading=h.gghzy4yds8t6)

[Table 5.3.2.1 Test Environment 239](#_heading=h.p1sg81n21m5m)

[Table 5.3.3.1 Test Milestones 239](#_heading=h.gddu4h6azamf)

[Figure 5.4.1.1 Unit Test Cases 240](#_heading=h.jek3by6ctaf)

[Figure 5.4.2.1 Other Test Cases 241](#_heading=h.e6p5j471coc8)

[Figure 5.5.1.1 Unit Test Cases Statistic 242](#_heading=h.423y8bf4ssi1)

[Figure 5.5.2.1 Other Test Cases Statistic 243](#_heading=h.d9mcq5haj3fi)

[Table 6.1.1 Deliverable Package 244](#_heading=h.n90k51k5ara)

[Table 6.2.1.1 System Requirements 244](#_heading=h.8jlik4t4mdgh)

[Figure 6.2.2.1.1.1 Flutter Pub Get Result 246](#_heading=h.sd8hgqkgwmn4)

[Figure 6.2.2.1.1.2 Role Selection Screen 247](#_heading=h.76tv13yyt4kc)

[Figure 6.2.2.1.2.1 Npm Install Result 248](#_heading=h.8x4fbmw7vk0o)

[Figure 6.2.2.1.2.2 Npm Run Build Result 248](#_heading=h.j6gurfjev5s4)

[Figure 6.2.2.1.2.3 Web Login Screen 249](#_heading=h.t4nlbq7gfjf0)

[Figure 6.3.2.1 Member Home Screen — SOS Button 256](#_heading=h.p3e4y79ig933)

[Figure 6.3.2.2 Emergency Tracking Screen 257](#_heading=h.47laehdmy0zw)

[Figure 6.3.2.3 Snake Identification Screen 258](#_heading=h.7tr727y5zdm1)

[Figure 6.3.2.4 Snake Confirmation Screen 260](#_heading=h.2792nkoy6ehn)

[Figure 6.3.2.5 Snake Selection by Location Screen 262](#_heading=h.m2s4vwb9zuzb)

[Figure 6.3.2.6 Symptom Report Screen 263](#_heading=h.7ac7rxxtxg0d)

[Figure 6.3.2.7 First Aid Steps Screen (Snake Species Identified) 265](#_heading=h.ofiqp11cm0xw)

[Figure 6.3.2.8 First Aid Steps Screen (General First Aid) 266](#_heading=h.zi7k931laiwc)

[Figure 6.3.2.9 Severity Assessment Screen 268](#_heading=h.a8ilo2osslmk)

[Figure 6.3.2.10 Case Details Screen 270](#_heading=h.2on6d621euyg)

[Figure 6.3.2.11 Cancel Emergency Confirmation Dialog 272](#_heading=h.iayofteco1lg)

[Figure 6.3.2.12 Activity Detail - With Cancelled Status (Member) 273](#_heading=h.dqqufacxhljf)

[Figure 6.3.2.13 Operator Dashboard 274](#_heading=h.me5fu1z14rjz)

[Figure 6.3.2.14 Emergency Case Detail — Pending Status (Operator View) 275](#_heading=h.qmxoio4qiezn)

[Figure 6.3.2.15 Emergency Case Detail — Verified Status (Operator View) 275](#_heading=h.rolbas8pr2y1)

[Figure 6.3.2.16 Assign Rescuer Screen 276](#_heading=h.l1ie7tk643ea)

[Figure 6.3.2.17 Emergency Case Detail — Assigned Status (Operator View) 277](#_heading=h.an1eaa3rh8du)

[Figure 6.3.2.18 Assignment Expired Notification (Operator View) 278](#_heading=h.4tog6af7eyyi)

[Figure 6.3.2.19 Mission Detail - SOS Screen 279](#_heading=h.xev5hqh0ve0o)

[Figure 6.3.2.20 Navigation Map Screen 281](#_heading=h.lhi60zjo0084)

[Figure 6.3.2.21 On-Scene support Screen 282](#_heading=h.q8x01rnnx9ye)

[Figure 6.3.2.22 Find Nearby Hospital 283](#_heading=h.d9d5wx4qq8w1)

[Figure 6.3.2.23 Navigation to Hospital Screen 284](#_heading=h.qw835ytal3um)

[Figure 6.3.2.24 Complete Support Mission 285](#_heading=h.qdx2jxnh2f2n)

[Figure 6.3.2.25 Mission Completion Screen 287](#_heading=h.sq7d321nsbtv)

[Figure 6.3.2.26 Mission Success - Emergency Screen 288](#_heading=h.x0h8zsji7270)

[Figure 6.3.2.27 Member Incident Finished Screen 289](#_heading=h.rj8829i11lp)

[Figure 6.3.2.28 Member Payment Selection 290](#_heading=h.fstsn7c6biyr)

[Figure 6.3.2.29 Payment Success 291](#_heading=h.38wrm7tj2vul)

[Figure 6.3.2.30 Activity History Screen (Member - tab “Sự cố”) 292](#_heading=h.4bcnpnq80d5f)

[Figure 6.3.3.1 Member Homepage — "Cần bắt rắn" Button 294](#_heading=h.nqfstgf4mk8m)

[Figure 6.3.3.2 Snake Quantity Service Selection Screen 295](#_heading=h.fu55ffqamu8e)

[Figure 6.3.3.3 Snake Report Detail Screen 296](#_heading=h.24tcrft78cn7)

[Figure 6.3.3.4 Snake Report Detail Screen (Upload Image for AI Snake Detection) 298](#_heading=h.rxi4bnugdeua)

[Figure 6.3.3.5 Snake Report Detail Screen (Snake AI Detection Result) 299](#_heading=h.9o6de4kkqc1q)

[Figure 6.3.3.6 Snake Report Detail Screen (Snake Selection by Location) 300](#_heading=h.93v1zg1qezak)

[Figure 6.3.3.7 Snake Report Detail Screen (Fill Full Request Form) 302](#_heading=h.60slvoj80stl)

[Figure 6.3.3.8 Snake Catching Success Screen 304](#_heading=h.pw4xwb6ou7x9)

[Figure 6.3.3.9 Member Payment Selection 305](#_heading=h.qpwrd7p6iphy)

[Figure 6.3.3.10 Payment Success For Travel Fee 306](#_heading=h.6swjw0f9gtvu)

[Figure 6.3.3.11 Member Cancel Request Button 308](#_heading=h.qx044wxt4cx6)

[Figure 6.3.3.12 Cancel Request Confirmation Dialog (Member) 309](#_heading=h.ay24nhcpsr0w)

[Figure 6.3.3.13 Operator Dashboard (“Bắt Rắn” Tab) 310](#_heading=h.izup6p7b6l25)

[Figure 6.3.3.14 Catching Case Detail — Pending Status (Operator View) 311](#_heading=h.ngo3zqbqdvsk)

[Figure 6.3.3.15 Catching Case Detail — Confirmed Status (Operator View) 311](#_heading=h.uw98zsj8r9xv)

[Figure 6.3.3.16 Assign Rescuer Screen 312](#_heading=h.3quohnqhjwjt)

[Figure 6.3.3.17 Catching Case Detail — Assigned Status (Operator View) 313](#_heading=h.pep3hv5cczu6)

[Figure 6.3.3.18 Request Catching Modal 314](#_heading=h.r5cmxn1n2jik)

[Figure 6.3.3.19 Accept Request Screen 315](#_heading=h.xud78jlvek3d)

[Figure 6.3.3.20 Rescuer Cancel Request Button 317](#_heading=h.pfuq2wccmin)

[Figure 6.3.3.21 Cancel Request Confirmation Dialog (Rescuer) 318](#_heading=h.pfp6fttgipm9)

[Figure 6.3.3.22 En-Route Screen 319](#_heading=h.pwnn8rywo2m9)

[Figure 6.3.3.23 Tracking Screen 320](#_heading=h.sq41ze46vwlz)

[Figure 6.3.3.24 Result Confirmation Screen 322](#_heading=h.2aigc2t9z0tj)

[Figure 6.3.3.25 Mission Success - Snake Catching Screen 323](#_heading=h.1xrnynyi8gc1)

[Figure 6.3.3.26 Member Payment Selection 324](#_heading=h.cvdt0cxfs50v)

[Figure 6.3.3.27 Payment Catching Success 325](#_heading=h.rcbtx9qkqdcp)

[Figure 6.3.3.28 Activity History Screen (Member - tab “Bắt rắn”) 326](#_heading=h.egk77ugq93ws)

[Figure 6.3.4.1 Expert Directory Screen 328](#_heading=h.crjux925wfwg)

[Figure 6.3.4.2 Expert Detail Screen 329](#_heading=h.4fzg2x8zbv6o)

[Figure 6.3.4.3 Schedule Selection Screen 330](#_heading=h.jiuenf236vbn)

[Figure 6.3.4.4 Consultation Detail Form Screen 332](#_heading=h.rg40mzfa5z46)

[Figure 6.3.4.5 Confirm & Payment Screen 333](#_heading=h.swcwrtk4sqts)

[Figure 6.3.4.6 Consultation Home Screen 334](#_heading=h.xpxnjstchnz7)

[Figure 6.3.4.7 Waiting Room Screen (Member) 335](#_heading=h.zmpgg1auwo5)

[Figure 6.3.4.8 Waiting Room Screen (Expert) 336](#_heading=h.5k79vg8czd6)

[Figure 6.3.4.9 Video Consultation Screen 337](#_heading=h.15pvpwbcky88)

[Figure 6.3.4.10 Rating Screen 339](#_heading=h.b89j8jyu2tg)

[Figure 6.3.5.1 Expert Detail Screen — Instant Consultation 341](#_heading=h.li7k8jmwkygo)

[Figure 6.3.5.2 Immediate Consultation Disabled 342](#_heading=h.t4dl97s2c22v)

[Figure 6.3.5.3 Consultation Detail Form Screen 343](#_heading=h.jza1okb4rqe9)

[Figure 6.3.5.4 Confirm & Payment Screen 345](#_heading=h.fk0qeg99xo8a)

[Figure 6.3.5.5 Waiting Expert Response Screen 346](#_heading=h.u78wxrn27g3v)

[Figure 6.3.5.6 Expert Global Emergency Popup Listener 348](#_heading=h.greztw1p6x6r)

[Figure 6.3.5.7 Member and Expert Waiting Room Screen 349](#_heading=h.el7182s2eul8)

[Figure 6.3.5.8 Video Consultation Screen 350](#_heading=h.j9i1ir3lgeng)

[Figure 6.3.5.9 Rating Screen 352](#_heading=h.vt6e9qenfn6a)

[Figure 6.3.5.1 Admin Dashboard 354](#_heading=h.qihc9xqfya0i)

[Figure 6.3.5.2 User Management Screen 355](#_heading=h.1v5gi33anthd)

[Figure 6.3.5.3 User Detail Screen 355](#_heading=h.yo8ieyvnnl14)

[Figure 6.3.5.4 Lock User Account Dialog 356](#_heading=h.6qekzmqkdfsd)

[Figure 6.3.5.5 Workshifts Management Screen 357](#_heading=h.nkkv0o1xuwtz)

[Figure 6.3.5.6 Shift Assignment Details Modal 358](#_heading=h.xfp79x91fydn)

[Figure 6.3.5.7 Incidents Management Screen 359](#_heading=h.1u9ml8xoehp2)

[Figure 6.3.5.8 Incident Detail Modal 359](#_heading=h.t6wdff4nngnu)

[Figure 6.3.5.9 Snake Catching Management Screen 360](#_heading=h.jbqhdtzct6n6)

[Figure 6.3.5.10 Snake Catching Detail Modal 360](#_heading=h.1m17xnb5wxrv)

[Figure 6.3.5.11 Consultations Management Screen 361](#_heading=h.hkbm5s7sb840)

[Figure 6.3.5.12 Consultation Detail Modal 361](#_heading=h.eish3xh1bajf)

[Figure 6.3.5.13 Snakes Management Screen 362](#_heading=h.2xykcmurrml6)

[Figure 6.3.5.14 Antivenoms Management Screen 363](#_heading=h.9ired4hfu79u)

[Figure 6.3.5.15 Treatment Facilities Management Screen 363](#_heading=h.s020upadzj75)

[Figure 6.3.5.16 Transactions & Withdrawals Management Screen 364](#_heading=h.xf4i06dsxfzn)

[Figure 6.3.5.17 Withdrawals Management Screen - “Đã Xử Lý” Tab 365](#_heading=h.tz218qmcrnda)

[Figure 6.3.5.18 Withdrawals Management Screen - “Đang Xử Lý” Tab 365](#_heading=h.nmczwkahh39w)

[Figure 6.3.5.19 Lessons Management Screen 366](#_heading=h.hc7u7qrpdr1s)

[Figure 6.3.5.20 Blogs Management Screen 367](#_heading=h.febqippobfak)

[Figure 6.3.5.21 Report Media Management Screen 368](#_heading=h.1q94r56tdegc)

[Figure 6.3.5.22 Settings Management Screen 369](#_heading=h.xzosbvp5o02u)

# Acknowledgement

We would like to express our deepest gratitude to FPT University for providing a professional academic environment and the essential resources that have fostered our growth throughout our journey.

Our most sincere thanks go to our supervisor, Mr. Do Tan Nhan, for his invaluable guidance, unwavering support, and insightful mentorship. His expertise and dedication were instrumental in helping us navigate technical challenges and elevating the overall quality of this project.

We are also profoundly grateful to Mr. Phan Minh Tam and Mr. Lam Huu Khanh Phuong for their time and expertise in evaluating our work. Their constructive feedback and critical insights have been vital in refining our final results.

Finally, we would like to thank our families and friends for their constant encouragement and belief in us. This achievement is as much theirs as it is ours.

# Definition and Acronyms

| **Acronym** | **Definition** |
| --- | --- |
| BA | Business Analysis |
| BR | Business Rule |
| ERD | Entity Relationship Diagram |
| GUI | Graphical User Interface |
| PM | Project Manager |
| SDD | Software Design Description |
| SPMP | Software Project Management Plan |
| SRS | Software Requirement Specification |
| UAT | User Acceptance Test |
| UC | Use Case |
| API | Application Program Interface |

###### Table 1.1 Definition and Acronyms

# I. Project Introduction

## 1. Overview

### 1.1 Project Information

* Project Name: AI-Powered Platform for Snakebite First Aid and Rescue Support
* Project Code: SP26SE001
* Group Name: GPSP26SE92
* Semester: Spring2026 (01/2026 - 05/2026)
* Software Type: Web Application, Mobile App

### 1.2 Project Team

| **Full Name** | **Role** | **Email** | **Mobile** |
| --- | --- | --- | --- |
| Đỗ Tấn Nhàn | Lecturer | nhandt35@fe.edu.vn | 0903056041 |
| Đoàn Ngọc Trung | Leader | trungdnse183494@fpt.edu.vn | 0787171600 |
| Phan Anh Khoa | Member | khoapase183495@fpt.edu.vn | 0336307487 |
| Nguyễn Văn Duy Khiêm | Member | khiemnvdse180168@fpt.edu.vn | 0836262507 |
| Nguyễn Phúc Nhân | Member | nhannpse184696@fpt.edu.vn | 0339206739 |
| Nguyễn Mạnh Dưỡng | Member | duongnmse181515@fpt.edu.vn | 0348018758 |

###### Table 1.1.2.1 Project’s member list

## 2. Product Background

Snakebite incidents are often poorly handled due to lack of timely first aid, misidentification of snake species, limited access to nearby antivenom facilities, and slow rescue coordination. Patients frequently panic and perform harmful treatments, while rescuers face risk without accurate species information. Experts are not always available for quick verification, and incident data is fragmented. Administrators lack real-time visibility of high-risk areas and rescue efficiency. Therefore, an AI-powered platform and coordination center integrating first-aid support, snake identification, rescue tracking, expert consultation, and incident monitoring is urgently needed.

## 3. Existing Systems

### 3.1 iNaturalist

iNaturalist is a biodiversity identification platform that allows users to upload images of animals and identify species using AI recognition combined with community contributions. This platform is useful for identifying snake species and learning about their characteristics, habitat, and potential risks. It helps users gain general awareness and supports educational purposes related to wildlife and snake identification. However, iNaturalist is not designed for emergency situations, as the identification process may take time and depends on community interaction. The platform also does not provide first-aid guidance, rescue coordination, or real-time expert consultation. SnakeAid aims to build upon these strengths by providing fast AI-based snake identification specifically designed for emergencies, along with immediate first-aid instructions and rescue coordination features.

Link: <https://www.inaturalist.org/>

### 3.2 Vietnam Snakes

Vietnam Snakes is an online resource dedicated to providing detailed information about snake species found in Vietnam. The platform allows users to explore various types of snakes, including their physical characteristics, habitats, and levels of danger (venomous or non-venomous). It is particularly useful for education and raising awareness, helping users better understand local snake biodiversity and how to identify different species. However, Vietnam Snakes mainly focuses on static reference content and does not offer interactive features such as symptom checking, real-time first-aid guidance, or emergency support. It also lacks AI-based snake identification and communication tools. SnakeAid enhances this by integrating intelligent snake recognition, personalized first-aid instructions, and real-time coordination with rescuers and experts.

Link: <https://vietnamsnakes.com/all-of-species>

### 3.3 Google Maps

Google Maps is a widely used navigation and location service that provides real-time tracking, route optimization, and nearby location search. These features are useful in emergency situations, allowing users to find nearby hospitals, clinics, and medical facilities quickly. Navigation and estimated arrival time also help rescuers reach victims more efficiently. However, Google Maps focuses mainly on navigation and does not provide snakebite-specific support such as snake identification, first-aid guidance, or rescue coordination. It also lacks incident monitoring and communication features between victims, rescuers, and administrators. SnakeAid enhances these capabilities by integrating location tracking with rescue coordination, enabling administrators to assign rescuers, monitor progress, and manage snakebite incidents in real time.

Link: <https://www.google.com/maps>

## 4. Business Opportunity

SnakeAid is a platform and coordination center that provides a comprehensive solution involving patients, rescuers, snake experts, and administrators. Patients receive step-by-step first-aid guidance, AI-based snake identification and severity assessment, SOS emergency calls with GPS sharing, real-time map tracking of rescue teams, symptom monitoring, and direct payments for consultations and rescue services. Snake rescuers receive alerts, manage rescue tasks, identify snake species via AI, navigate to incident locations using map-based guidance, and update status. Snake experts verify images, provide remote consultation, update treatment guidelines, and manage earnings. Admins oversee user roles, snake database, treatment facilities, content, system analytics, map-based activity monitoring, alerts, and platform-wide financial reports.

## 5. Software Product Vision

SnakeAid aims to become the most reliable and comprehensive snakebite first-aid platform and rescue coordination center. It provides a unified and real-time ecosystem connecting patients, rescuers, snake experts, and administrators. AI technology is used for snake identification while the coordination center handles rescue assignment, communication, and incident monitoring to ensure faster and more effective emergency response.

Our vision is to transform the way snakebite incidents are handled by delivering:

* **Faster life-saving response** – Provide immediate, step-by-step first-aid guidance to reduce panic and prevent harmful traditional treatments.
* Accurate identification and decision support – Utilize AI-powered snake species recognition and severity assessment to support appropriate medical and rescue actions.
* **Right connection at the right time** – Automatically dispatch the nearest rescue team, enable GPS location sharing, and provide real-time rescue tracking.
* **Standardized expert validation** – Enable snake experts to verify species, provide remote consultations, and continuously update treatment guidelines.
* **Data-driven management and prevention** – Equip administrators with real-time dashboards, hotspot mapping, rescue performance analytics, and transparent financial reporting.

In the long term, SnakeAid aspires to become a smart healthcare and rescue infrastructure platform, reducing mortality and complications caused by snakebites, improving public awareness, and optimizing rescue resources in high-risk regions.

## 6. Project Scope & Limitations

### 6.1 Project Scope

**FE-01: Authentication & Authorization**

* Sign Up (Member, Expert roles)
* Sign In with Email/Password
* Sign Out & Session Cleansing
* Verify Email Identity (via OTP)
* Reset Password via Secure Token

**FE-02: User Profile Management**

* Get My Profile Information
* Update Profile (Name, Bio, Emergency Contacts)
* Upload & Crop Profile Avatar
* Get User Activity Logs (Login history)

**FE-03: Professional Onboarding (Expert-side)**

* Upload Certification & Identity Documents
* Complete Expert Registration Profile
* Submit Verification Request
* Track Verification Status
* Toggle Real-time Availability (Active/Inactive)

**FE-04: AI Image Recognition Engine**

* Capture Snake Photo with Camera Interface
* Image Pre-processing (Crop, Blur Detection, Brightness)
* Return Probability Scores & Top Matches

**FE-05: Snake Species Knowledge Base**

* Get Detailed Species Information (Scientific Name, Local Name)
* View Toxicity Level & Danger Assessment
* Display Habitat & Distribution Maps

**FE-06: Severity Assessment**

* Process Symptom Inputs (Pain Level, Nausea, Breathing)
* Categorize Severity (Mild, Moderate, Severe, Critical)
* Generate Immediate Medical Recommendation

**FE-08: Emergency SOS Trigger**

* Hold SOS Emergency Activation
* Capture Instant GPS Coordinates (Latitude/Longitude)
* Create Emergency Case and Push to Operator Queue

**FE-09: Medical Facility Intelligence**

* Identify Nearest Hospitals
* Filter Facilities by Distance and Availability
* Calculate Estimated Time of Arrival (ETA)
* Integrated Navigation

**FE-10: Real-time Rescue Tracking (SSE/Socket)**

* Connect to Real-time Location Stream
* Broadcast Rescuer Movement on Member Map
* Display Route Polyline & Dynamic ETA Updates
* Broadcast Arrival & Task Completion Events

**FE-11: Snake Catching Service**

* Create Non-emergency Catching Request (Photo, Location, Description)
* Automatic Price Estimation
* Track Mission Sub-statuses

**FE-12: Expert Consultation (Remote)**

* Create Real-time Chat Session with Snake Expert
* Start Peer-to-Peer Video Call for Triage
* Share High-resolution Media for Visual Diagnosis
* End Session & Consultation Summary

**FE-13: Rescuer Mission Management (Operator-assigned)**

* Receive SignalR Dispatch Alerts from Operator
* Acknowledge Assignment Reception
* Update Mission Status
* View Mission History & Performance

**FE-14: Safety Guidelines & Knowledge Hub**

* Access Step-by-step First Aid Instructions
* View Compression Bandaging Video Tutorials
* Read Prevention Articles & FAQ
* Search Knowledge Base by Topic/Snake Group

**FE-15: Service Pricing & Fee Configuration**

* Set Expert Consultation Rates
* Configure Service Fee
* Define Platform Commission Percentages

**FE-16: Payment Integration & Processing (PayOS)**

* Checkout via PayOS Gateway
* Process Payment Flow by Service Type
* Process Service Fee Refunds (for Valid Cancellations)

**FE-17: Revenue Tracking (User-facing)**

* Get Monthly Revenue Reports (Expert)
* Calculate Final Payouts After Platform Fees
* View Transaction History with Filtering

**FE-18: Operational Dispatching (Operator)**

* View Real-time Emergency & Snake Catching Request Queues
* Verify & Triage Incoming Reports
* Manual Assignment of Rescuers to Specific Cases

**FE-19: Identity & Professional Verification (KYC Admin-side)**

* Review Professional Documentation (Certificates)
* Approve/Decline Expert Registration Requests
* Manage Professional Badges and Verification Status
* Suspend/Re-activate Accounts Based on Performance/Compliance

**FE-20: Hospital & Antivenom Management (Admin)**

* Create/Update/Delete Hospital Records
* Update Antivenom Stock & Availability
* Tag Facilities with Specialty (e.g., 24/7 Treatment)
* Import Hospital Data from External Sources

**FE-21: Snake Database Administration**

* Manage Master Snake Species List
* Upload Training Data for AI Model (Admin Review)
* Manage Toxicity Classifications & Symptoms Metadata
* Link Species to Specific Antivenom Types

**FE-22: System Content Management (Admin)**

* Update First Aid Guidelines and Snake Knowledge Content
* Manage Blogs for Member
* Manage Lessons for Rescuer

**FE-23: Smart Notification Orchestration**

* Push Notifications for Services
* In-app Alerts for System Updates
* Role/Area-based Notification Campaign Rules

**FE-24: Feedback & Rating System**

* Rate Rescuer Performance (Member)
* Rate Expert Consultation Quality (Member)
* Leave Review/Comment on Services

**FE-25: Platform Revenue & Commission Management (Admin)**

* Track Total Revenue and Distribute Income
* Manage Platform Commission and Payout Schedules
* Handle Payment Disputes and Refund Requests

**FE-26: Wallet Management (SnakeAidPay)**

* Top-up Wallet Balance (PayOS)
* Withdraw to Linked Bank Account
* View Wallet Balance & Available Amount
* View Wallet Transaction History
* Role-based Wallet Access Rules

**FE-27: PayOS Callback & Payment Reconciliation**

* Handle Deep Link Callback (Success/Cancel/Failed)
* Validate Callback Integrity (orderCode, amount, signature)
* Idempotent Callback Processing
* Auto Retry + Confirm Payment Fallback
* Sync Final Payment Status to Booking/Wallet/Invoice

**FE-28: Consultation Booking Lifecycle & Escrow Control**

* Manage Booking States
* Create Waiting Room
* Auto Lock/Release Expert Time Slot
* Escrow Hold on Confirmed Booking
* Escrow Release/Refund by Cancellation Policy

**FE-29: Catching Service Two-Phase Payment Control**

* Round 1 Travel Fee Payment
* Round 2 Final Service Payment
* Deposit Usage Rules by Status
* Cancellation Window & Penalty Rules

**FE-30: Notification Center & Real-time Reliability**

* In-app Notification Inbox
* Realtime Mission Events via SignalR/SSE
* Connection Reconnect on App Resume/Network Change
* User-level Notification Preferences

**FE-31: Audit, Compliance & AI Governance**

* Log Expert Override for AI Identification Results
* Track Decision History for Severity Recommendations
* Risk Rules for Low-confidence AI Outputs (Fallback to Safe Protocol)

### 6.2 Limitations & Exclusions

**Limitations (LI):**

**LI-1:** The platform does not replace professional medical treatment and shall not be considered a substitute for hospital care or licensed healthcare services.

**LI-2:** AI-based snake identification and severity assessment may not achieve 100% accuracy, especially with low-quality images or incomplete symptom data. Final medical decisions remain the responsibility of healthcare professionals.

**LI-3:** The system does not guarantee the availability of rescue teams or snake experts in all geographic areas at all times. Service coverage depends on registered personnel in the region.

**LI-4:** The platform requires stable internet connectivity for real-time tracking, image upload, AI processing, and online payment. Offline functionality is limited to previously loaded static content (if available).

**LI-5:** The system does not directly manage hospital operations, ambulance dispatch systems, or government emergency infrastructure. Integration with public emergency systems is outside the project scope.

**LI-6:** The platform does not provide physical snake removal equipment, medical supplies, or antivenom stock management beyond informational display.

**LI-7:** The project does not include hardware development such as wearable medical devices, IoT tracking devices, or dedicated emergency communication equipment.

**LI-8:** The system will not include international regulatory compliance beyond the initially targeted deployment region during the project phase.

**LI-9:** The platform does not guarantee financial compensation for unsuccessful rescue attempts or medical outcomes. Payment covers service effort rather than treatment results.

**LI-10:** The project does not include advanced predictive analytics for nationwide epidemiological forecasting beyond basic statistical reporting and hotspot visualization.

**Exclusions (EX):**

**EX-1:** Offline-First & Data Synchronization:The system requires active internet connectivity for all operations. Offline mode, local data caching, and delayed sync patterns are excluded; real-time connectivity is mandatory for SOS, payments, and expert consultations.

**EX-2:** SMS, Voice Calls & VoIP Integration:The system uses in-app notifications and SignalR real-time events exclusively. SMS messaging, phone calls, voice-based emergency dispatch, and VoIP are excluded; video consultation with Expert is the only communication channel outside push notifications.

**EX-3:** Multi-Language Localization (Phase 1):Initial release supports Vietnamese language only. i18n frameworks, RTL layouts, and multi-language content management are excluded; future language expansion is a separate initiative.

**EX-4:** Blockchain, Cryptocurrency & Non-Traditional Payments**:** The payment system uses PayOS (Vietnam traditional gateway) exclusively. Blockchain, cryptocurrency wallets, DeFi channels, and decentralized payment mechanisms are excluded; all transactions must flow through regulated Vietnamese payment infrastructure.

**EX-5:** Multi-Tenant & SaaS Platform:The platform is single-tenant (SnakeAid Vietnam only). Multi-tenant architecture, white-label licensing, partner portals, and per-organization configuration are excluded.

**EX-6:** Third-Party Messaging & Chat Integration: The system uses native in-app consultation (video + text chat widget) only. Integration with WhatsApp, Telegram, Messenger, Zalo, or other messaging platforms is excluded.

**EX-7:** Hardware Integration & IoT Devices**:** The platform does not integrate with wearable devices, GPS trackers, sensors, or specialized rescue equipment. Location tracking is mobile GPS only; hardware integrations are excluded.

**EX-8:** Legacy Data Migration & SnakeAid v1.0 Compatibility: The system launches as a clean slate. Historical data from SnakeAid v1.0, backward-compatible APIs, and legacy system bridges are excluded.

# II. Project Management Plan

## 1. Overview

### 1.1 Scope & Estimation

| **#** | **WBS Item** | **Complexity** | **Est. Effort**  **(man-days)** |
| --- | --- | --- | --- |
| **1** | **Feature 1 — Authentication & Account Management** |  | **8** |
| 1.1 | Login / Logout (Member, Rescuer, Expert, Operator) | Simple | 2 |
| 1.2 | Member registration & OTP verification | Simple | 3 |
| 1.3 | Personal profile, avatar & security management | Medium | 3 |
| **2** | **Feature 2 — SOS Emergency Response (Snakebite)** |  | **30** |
| 2.1 | Send SOS request: auto GPS, snake photo, symptom description | Medium | 6 |
| 2.2 | Operator verifies SOS & assigns Rescuer via SignalR | Complex | 9 |
| 2.3 | Rescuer receives mission, updates movement status | Medium | 8 |
| 2.4 | SOS service payment (PayOS / SnakeAid Wallet) & case closure | Medium | 7 |
| **3** | **Feature 3 — Snake Catching Request** |  | **12** |
| 3.1 | Create snake catching request: photo, GPS, species & quantity description | Simple | 2 |
| 3.2 | Round 1 deposit payment — CatchingDeposit (PayOS / SnakeAid Wallet) | Medium | 3 |
| 3.3 | Operator validates request & assigns suitable Rescuer | Medium | 2 |
| 3.4 | Rescuer completes mission, records results & photo evidence | Medium | 3 |
| 3.5 | Round 2 payment — CatchingPayment (actualCost) & order completion | Medium | 2 |
| **4** | **Feature 4 — Expert Consultation** |  | **15** |
| 4.1 | Expert listing & profile view | Simple | 2 |
| 4.2 | Instant consultation (SignalR) | Complex | 4 |
| 4.3 | Scheduled consultation | Medium | 3 |
| 4.4 | Video call, chat & escrow payment | Complex | 6 |
| **5** | **Feature 5 — AI Snake Identification & First-Aid Knowledge** |  | **15** |
| 5.1 | Upload photo & AI snake identification | Medium | 5 |
| 5.2 | Display toxicity & danger level | Simple | 3 |
| 5.3 | First-aid guideline & snake information lookup | Medium | 4 |
| 5.4 | Nearest hospital / treatment center map | Medium | 3 |
| **6** | **Feature 6 — SnakeAid Wallet & Payment System** |  | **16** |
| 6.1 | Wallet overview & balance | Simple | 2 |
| 6.2 | Top-up via PayOS | Complex | 5 |
| 6.3 | Withdraw to bank account | Medium | 5 |
| 6.4 | Transaction history | Simple | 4 |
| **7** | **Feature 7 — Notifications & Realtime System** |  | **7** |
| 7.1 | Push Notification (FCM) | Medium | 3 |
| 7.2 | SignalR realtime updates | Complex | 4 |
| **8** | **Feature 8 — Rescuer & Expert Mobile Applications** |  | **16** |
| 8.1 | Rescuer mission management | Medium | 5 |
| 8.2 | Expert consultation management | Medium | 5 |
| 8.3 | Map navigation & realtime tracking | Medium | 6 |
| **9** | **Feature 9 — Operator Web Application** |  | **6** |
| 9.1 | SOS / Snake catching queue | Medium | 2 |
| 9.2 | Assign rescuer / expert | Medium | 2 |
| 9.3 | Realtime status monitoring | Medium | 2 |
| **10** | **Feature 10 — Admin Web Application** |  | **14** |
| 10.1 | Dashboard & system statistics | Simple | 2 |
| 10.2 | User & role management | Medium | 2 |
| 10.3 | Case management (SOS / Catching / Consultation) | Complex | 2 |
| 10.4 | Snake species management | Medium | 2 |
| 10.5 | Hospital & treatment center management | Medium | 2 |
| 10.6 | Transaction management | Medium | 3 |
| 10.7 | Lesson & Article management | Simple | 1 |
| **Total Estimated Effort (man-days)** | | | **132** |

###### Table 2.1.1.1 Scope & Estimation

### 1.2 Project Objectives

| **#** | **Testing Stage** | **Test Coverage** | **No. of Defects** | **% of Defect** | **Notes** |
| --- | --- | --- | --- | --- | --- |
| 1 | Reviewing | 100% core modules | < 12 | 12% | Code review |
| 2 | Unit Test | ≥ 80% | < 10 | 10% | Core logic |
| 3 | Integration Test | ≥ 70% | < 8 | 8% | API flow |
| 4 | System Test | ≥ 90% | < 5 | 5% | End-to-end |
| 5 | Acceptance Test | 100% critical | 0 critical | 0% | Client verify |

###### Table 2.1.2.1 Project Objectives

**Milestone Timelines** (%): 95% on-time delivery

**Allocated Effort** (man-days): 180 man-days

### 1.3 Project Risks

| **#** | **Risk Description** | **Impact** | **Possibility** | **Response Plans** |
| --- | --- | --- | --- | --- |
| 1 | LiveKit / WebRTC integration for Video Call is complex and depends on server configuration | High | Medium | Prioritize early POC in Week 1–2 and prepare fallback to simpler video solution if needed |
| 2 | PayOS deep link callback on android (Universal Link) may encounter issues | Medium | Medium | Test on real devices early and prepare manual callback handling as fallback |
| 3 | Snake AI recognition API may change response format | Low | Medium | Wrap AI integration inside a separate service layer to allow easy replacement or adjustment |
| 4 | SignalR connection may be unstable on weak network conditions | Medium | High | Implement auto reconnect logic, timeout handling, and fallback UI for offline scenarios |
| 5 | Backend may not provide real-time GET status endpoints for some flows | Medium | Medium | Confirm API contract with Backend team before Week 2 and define fallback polling mechanism |

###### Table 2.1.3.1 Project Risks

## 2. Management Approach

The SnakeAid project will adopt an **Agile development approach** to ensure flexibility, continuous improvement, and early delivery of key features. Agile allows the team to adapt to requirement changes, reduce risks, and continuously improve system quality.

The project will be divided into multiple **iterations (sprints)**, where each sprint includes planning, design, development, testing, and review activities. Regular meetings and progress tracking will be conducted to ensure timely delivery and maintain project quality.

The Agile approach is selected because the SnakeAid system includes multiple complex components such as:

* SOS emergency response
* AI snake identification
* Real-time communication
* Payment and wallet system
* Mobile and web applications

These components require iterative development and continuous validation.

### 2.1 Project Process

The project will follow an **Agile Iterative Development Process** as shown below:

*![](data:image/jpeg;base64...)*

###### Figure 2.2.2.1 Agile Scrum Cycle

Each sprint will typically last **1–2 weeks**, and the team will deliver incremental features after each sprint.

**Agile Process Description:**

**Sprint Planning** - During Sprint Planning, the team reviews the product backlog and selects the features to be implemented in the upcoming sprint. The team defines sprint objectives, estimates effort, assigns responsibilities, and determines the deliverables for the sprint. This activity ensures that all team members clearly understand the scope and priorities before development begins.

**Requirement Analysis** - In this phase, the team analyzes the selected features in detail. Functional requirements, user flows, and technical constraints are clarified. The team also identifies dependencies, risks, and acceptance criteria to ensure that all requirements are well understood before implementation.

**System Design** - During the System Design phase, the team designs the system architecture, database structure, APIs, and UI components. Technical decisions such as technology selection, data flow, and integration approaches are finalized. This phase ensures a solid foundation for development and reduces technical risks.

**Development** - In the Development phase, team members implement the assigned tasks according to the system design. Developers follow coding standards, implement core features, and integrate components. Regular communication is maintained to ensure progress and resolve technical issues.

**Testing** - Testing is conducted during and after development. Unit testing is performed to validate individual components, while integration testing ensures proper communication between system modules. Bugs and defects are recorded and fixed during the sprint.

**Sprint Review** - During Sprint Review, the team demonstrates completed features to stakeholders or supervisors. Feedback is collected, and any necessary adjustments are identified. This activity ensures that the project is aligned with expectations and requirements.

**Sprint Retrospective** - After Sprint Review, the team conducts a Sprint Retrospective to evaluate the sprint performance. The team discusses what went well, what issues occurred, and what improvements can be made. Lessons learned are applied to improve the next sprint.

### 2.2 Quality Management

To ensure high system quality, the team will implement the following quality management approaches:

**Defect Prevention**

The team will conduct requirement clarification and system design review before implementation to reduce defects early in the development phase.

**Reviewing**

Code reviews will be conducted for core modules to ensure coding standards, reduce logic errors, and improve maintainability.

**Unit Testing**

Unit testing will be performed for core business logic such as:

* SOS request handling
* Payment processing
* AI snake identification
* Wallet transaction

**Integration Testing**

Integration testing will validate system components including:

* Mobile app and backend APIs
* SignalR real-time communication
* Payment integration (PayOS)
* AI service integration

**System Testing**

System testing will verify complete end-to-end workflows including:

* SOS emergency response
* Snake catching request
* Expert consultation
* Wallet and payment system
* Admin and operator workflows

**Additional Quality Activities**

The team will also perform:

* Weekly sprint demo
* Bug tracking and resolution
* Continuous testing during development
* Final acceptance testing before deployment

### 2.3 Training Plan

| **Training Area** | **Participants** | **When, Duration** | **Waiver Criteria** |
| --- | --- | --- | --- |
| Agile & Sprint Workflow | All Team Members | Week 1, 1 day | Mandatory |
| Flutter Mobile Development | TrungDN, DuongNM, KhoaPA | Week 1, 3 days | Mandatory |
| Supabase | All Team Members | Week 1, 1 day | Mandatory |
| Docker - Docker  Compose | Backend Developers | Week 2, 1 day | Mandatory |
| SignalR Realtime Communication | All Team Members | Week 2, 1 day | Mandatory |
| Fine-tuning AI | KhiemNVD, DuongNM | Week 3, 5 days | Mandatory |

###### Table 2.2.3.1 Training Plan

## 3. Project Deliverables

| **#** | **Deliverable** | **Due Date** | **Notes** |
| --- | --- | --- | --- |
| 1 | Software Requirement  Specification, Business  Rules. | Week 3 | The deliverable provides clear functional and non-functional features. Business rules are defined |
| 2 | Application Prototype | Week 5 | Design low-fidelity prototype and complete database schema |
| 3 | Core Features (phase 1) | Week 6 | Authentication, Wallet, Basic API development |
| 4 | Core Features (phase 2) | Week 8 | SOS, Snake Catching, Expert Consultation features implementation |
| 5 | Integration Testing | Week 11 | API integration and service interaction testing |
| 6 | System Testing | Week 13 | End-to-end workflow testing and bug fixing |
| 7 | Deployment | Week 14 | System deployment to production environment |
| 8 | Complete Documents | Week 15 | Final report, user guide, technical documentation |

Table 2.3.1 Project Deliverables

##

## 4. Responsibility Assignments

*D~Do; R~Review; S~Support; I~Informed; <blank>- Omitted*

| **Responsibility** | **TrungDNSE183494** | **KhoaPASE183495** | **KhiemNVDSE180168** | **DuongNMSE181515** | **NhanNPSE184696** |
| --- | --- | --- | --- | --- | --- |
| Project Planning & Tracking | D | S | R | D | R |
| Prepare Project Introduction Document | R | S | R | R | D |
| Prepare SRS Document (Overview Part) | D | D | D | D | D |
| Prepare SRS Document (User Requirements) | D | D | D | D | D |
| Daily Stand-ups | D | R | R | R | R |
| Development of Features | D | D | D | D | D |
| Testing & Quality Assurance | I | S | D | D | D |
| Release Planning | D | S | D | D | S |

###### Table 2.4.1 Responsibility Assignments

## 5. Project Communications

##

| **Communication Item** | **Who/ Target** | **Purpose** | **When, Frequency** | **Type, Tool, Method(s)** |
| --- | --- | --- | --- | --- |
| Supervisor meetings | Supervisors  and team  members | Business Requirements review  Application main flows demonstration  Advice on features  Progress assurance | Every week, once or twice a week | Zalo, Offline |
| Teamwork meeting | Team  members | Discuss tasks progress, assign work, resolve blockers, coordinate development | Daily or 2-3 times/week (depending on sprint) | Zalo, Discord |
| Issue reporting | Team members | Report bugs, blockers, or system issues | As needed (real-time) | Zalo, Discord |

###### Table 2.5.1 Project Communication

## 6. Configuration Management

### 6.1 Document Management

* **Document**: use Google Docs for documentation, Google Sheets for unit tests and test reports. Both Google Docs and Google Sheets provide real-time edit, edit history.
* **Diagrams**: use draw.io for diagram visualization and Google Drive for storing diagrams.

### 6.2 Source Code Management

* **Version Control** - Git is used for local code version control because it is open-source and reliable in production for a long period.
* **Code storage** - GitHub is used for online and remote code storage because it is built based on Git and a powerful platform.
* **Source Code Management** - Frontend and Backend Services are stored in different GitHub repositories for seamless development between services.

### 6.3 Tools & Infrastructures

| **Category** | **Tools / Infrastructure** |
| --- | --- |
| **Technology** | Flutter (Mobile), NextJS (Frontend), ASP.NET Core (Backend), SignalR (Real-time Communication), PostgreSQL (Database), PostGIS (Geospatial Data), Redis & Upstash (Caching), FastAPI (AI Inference Serving), YOLO (AI Foundation Models), PyTorch(AI Fine-tuning framework) Roboflow (Dataset labeling) HuggingFace (AI Model and dataset registry), Jenkins & GitHub Action (CI/CD), CodeMagic (Mobile DevOps), Docker (Containerization), Cloudflare (Security/CDN), Doppler (Secret Management), xUnit (Testing), PayOS (Payment Gateway), Resend (Email Service), Cloudinary (Media Management), LiveKit (WebRTC), LocationIQ (Location Service), ZimaOS (Self-hosting Backend), Vercel (Web Hosting), GitHub & GitLab (Version Control). |
| **Database** | PostgreSQL (Supabase) |
| **IDEs/Editors** | Visual Studio Code, Visual Studio 2022 |
| **Diagramming** | [Draw.io](http://draw.io), Plant UML |
| **Documentation** | Google Docs, Google Sheets, Google Drive, Canva |
| **Version Control** | GitLab (Source Codes), Google Docs (Documents) |
| **Deployment server** | Docker, Self-hosted ZimaOS |
| **Project management** | Zalo, Google sheet |

###### Table 2.6.3.1 Tools & Infrastructures

#

#

#

#

#

# III. Software Requirement Specification

## 1. Product Overview

SnakeAid is a new AI-powered software platform that replaces the current fragmented and manual processes for handling snakebite incidents, which often rely on phone calls, informal communication, and unverified information sources. The system centralizes first-aid guidance, snake species identification, rescue coordination, expert consultation, and payment management into a unified digital solution. The context diagram below illustrates the external entities and system interfaces for Release 1.0, including patients, snake rescuers, snake experts, administrators, AI identification services, map and GPS services, payment gateways, and healthcare facilities. The system is expected to evolve over multiple releases, potentially expanding geographic coverage, improving AI capabilities, and integrating with external emergency and healthcare systems.

![](data:image/png;base64...)

Figure 3.1.1 Context Diagram

##

##

##

##

##

## 2. User Requirements

### 2.1 Actors

| **#** | **Actor** | **Description** |
| --- | --- | --- |
| 1 | Member | The end user of the system (victim or snake spotter). A Member can request first-aid support in case of snakebite, report incidents or request snake catching services, provide images and location data, track request status, receive guidance from the system/AI/experts, and make payments for services. |
| 2 | Operator | The central coordination staff (center-side). The Operator is responsible for receiving requests from Members (including both snakebite incidents and snake catching cases), performing initial assessment, assigning appropriate Rescuers, monitoring task progress, and ensuring effective coordination among all parties. This is a newly introduced role in the half-center half-platform model. |
| 3 | Rescuer | The field execution personnel. Rescuers receive assigned tasks from the Operator, travel to the incident location to handle snakebite emergencies or snake catching, update task status (en route, completed), and record mission-related data. Rescuers do not self-assign tasks but strictly act based on Operator assignments. |
| 4 | Expert | A domain expert in snakes. The Expert verifies snake species based on images or descriptions, provides remote consultation for Members, updates first-aid and treatment guidelines. This role remains unchanged from the original model. |
| 5 | Admin | The system administrator. Admin manages the entire platform, including users and roles, snake database, treatment facilities, content, system monitoring, analytics, reporting, and financial operations. |
| 6 | Payment Gateway | A third-party financial service provider that handles secure payment processing for the system. The Payment Gateway is responsible for processing transactions made by Members (e.g., service fees, emergency support payments), validating payment details, ensuring transaction security, and returning payment status (success/failure) to the system. It operates externally and does not store sensitive user data within the platform. |
| 7 | AI Service | An intelligent subsystem (or external service) that provides automated analysis of snake-related data. The AI Service processes images submitted by Members to identify snake species. |

###### Table 3.2.2.1 Actors

### 2.2 Use cases

#### 2.2.1 Diagram

![](data:image/png;base64...)

###### Figure 3.2.2.1.1 Use Case Diagram

#### 2.2.2 Description

| **#** | **Use Case** | **Actors** | **Use Case Description** |
| --- | --- | --- | --- |
| 1 | View snake | Member | Member can view snake. |
| 2 | Assign rescuer | Operator | Operator can assign rescuer. |
| 3 | View rescue mission | Rescuer | Rescuer can view rescue mission. |
| 4 | View catching mission | Rescuer | Rescuer can view catching mission. |
| 5 | Accept request | Rescuer | Rescuer can accept request. |
| 6 | Decline request | Rescuer | Rescuer can decline request. |
| 7 | Update mission status | Rescuer | Rescuer can update mission status. |
| 8 | Upload mission report | Rescuer | Rescuer can upload mission report. |
| 9 | View workshift | Operator, Rescuer | Operator and Rescuer can view workshift. |
| 10 | View nearby treatment facility | Member, Rescuer | Member and Rescuer can view nearby treatment  facility. |
| 11 | View others profile | Authorized User | Authorized User can view others profile. |
| 12 | Book consultation slot | Member | Member can book consultation slot. |
| 13 | Track rescuer's location | Member | Member can track rescuer's location. |
| 14 | Track snakebite symptom | Member | Member can track snakebite symptom. |
| 15 | View AI result | Member | Member can view AI result. |
| 16 | Scan AI snake image | Member | Member can scan AI snake image. |
| 17 | Join consultation room | Expert, Member | Expert and Member can join consultation room. |
| 18 | Report snakebite incident | Member | Member can report snakebite incident. |
| 19 | Request snake catching | Member | Member can request snake catching. |
| 20 | Manage schedule | Expert | Expert can manage schedule. |
| 21 | Update consultation fee | Expert | Expert can update consultation fee. |
| 22 | View consultation booking | Expert, Member | Expert and Member can view consultation booking. |
| 23 | Confirm consultation booking | Expert | Expert can confirm consultation booking. |
| 24 | Upload consultation report | Member | Member can upload consultation report. |
| 25 | Feedback | Member | Member can provide feedback. |
| 26 | Create slot | Expert | Expert can create slot. |
| 27 | Update slot | Expert | Expert can update slot. |
| 28 | View transaction history | Authorized User | Authorized User can view transaction history. |
| 29 | View slot | Expert, Member | Expert and Member can view slot. |
| 30 | View profile | Authorized User | Authorized User can view profile. |
| 31 | Update profile | Authorized User | Authorized User can update profile. |
| 32 | View first aid guideline | Authorized User | Authorized User can view first aid guideline. |
| 33 | View safety guidance | Authorized User | Authorized User can view safety guidance. |
| 34 | Create community report | Member | Member can create community report. |
| 35 | Delete community report | Member | Member can delete community report. |
| 36 | View app notification | Authorized User | Authorized User can view app notification. |
| 37 | Manage blog | Administrator, Expert, Member | Administrator, Expert, and Member can manage  blog. |
| 38 | Manage education content | Administrator | Administrator can manage education content. |
| 39 | Manage snake library | Administrator | Administrator can manage snake library. |
| 40 | Manage first aid guideline | Administrator | Administrator can manage first aid guideline. |
| 41 | Manage snake | Administrator | Administrator can manage snake. |
| 42 | Manage antivenom | Administrator | Administrator can manage antivenom. |
| 43 | Manage venom type | Administrator | Administrator can manage venom type. |
| 44 | Manage treatment facility | Administrator | Administrator can manage treatment facility. |
| 45 | Manage fee and revenue | Administrator | Administrator can manage fee and revenue. |
| 46 | View blog | Administrator, Expert, Member | Administrator, Expert, and Member can view blog. |
| 47 | Create blog | Administrator, Expert | Administrator, Expert can create blog. |
| 48 | Update blog | Administrator, Expert | Administrator, Expert can update blog. |
| 49 | Delete blog | Administrator, Expert | Administrator, Expert can delete blog. |
| 50 | View education content | Authorized User | Authorized User can view education content. |
| 51 | Manage lesson | Administrator | Administrator can manage lesson. |
| 52 | Create lesson | Administrator | Administrator can create lesson. |
| 53 | Update lesson | Administrator | Administrator can update lesson. |
| 54 | Delete lesson | Administrator | Administrator can delete lesson. |
| 55 | Create first aid guideline | Administrator | Administrator can create first aid guideline. |
| 56 | Update first aid guideline | Administrator | Administrator can update first aid guideline. |
| 57 | Delete first aid guideline | Administrator | Administrator can delete first aid guideline. |
| 58 | Create treatment facility | Administrator | Administrator can create treatment facility. |
| 59 | Update treatment facility | Administrator | Administrator can update treatment facility. |
| 60 | Delete treatment facility | Administrator | Administrator can delete treatment facility. |
| 61 | Update fee | Administrator | Administrator can update fee. |
| 62 | View statistic | Administrator | Administrator can view statistic. |
| 63 | Create snake | Administrator | Administrator can create snake. |
| 64 | Update snake | Administrator | Administrator can update snake. |
| 65 | Delete snake | Administrator | Administrator can delete snake. |
| 66 | Create antivenom | Administrator | Administrator can create antivenom. |
| 67 | Update antivenom | Administrator | Administrator can update antivenom. |
| 68 | Delete antivenom | Administrator | Administrator can delete antivenom. |
| 69 | Process gateway payment | Payment Gateway | Payment Gateway can process gateway payment. |
| 70 | Manage profile | Authorized User | Authorized User can manage profile. |
| 71 | Identify Snake Species | AI Service | AI Service can identify snake species. |
| 72 | request withdraw | Expert, Member | Expert and Member can request withdraw. |
| 73 | Topup wallet | Member | Member can top up wallet. |
| 74 | View wallet | Authorized User | Authorized User can view wallet. |
| 75 | View Feedback | Expert, Rescuer | Expert and Rescuer can view feedback. |
| 76 | Manage community report | Member | Member can manage community report. |
| 77 | Update community report | Member | Member can update community report. |
| 78 | View requests | Member | Member can view requests. |
| 79 | View Lesson | Rescuer | Rescuer can view lesson. |
| 80 | Make payment | Member | Member can make payment. |
| 81 | Pay via wallet | Member | Member can pay via wallet. |
| 82 | View community report | Member | Member can view community report. |
| 83 | Update status requests | Operator | Operator can update status requests. |

###### Table 3.2.2.2.1 Use Case Description

## 3. Functional Requirements

### 3.1 System Functional Overview

#### 3.1.1 Screens Flow

##### **3.1.1.1 Member Screen Flow**

![](data:image/png;base64...)

###### Figure 3.3.1.1.1.1 Member Screen Flow

##### **3.1.1.2 Rescuer Screen Flow**

![](data:image/png;base64...)

###### Figure 3.3.1.1.1.2 Rescuer Screen Flow

##### **3.1.1.3 Expert Screen Flow**

***![](data:image/png;base64...)***

###### Figure 3.3.1.1.1.3 Expert Screen Flow

#####

##### **3.1.1.4 Admin and Operator Screen Flow**

*![](data:image/png;base64...)*

###### Figure 3.3.1.1.4 Admin and Operator Screen Flow

#### 3.1.2 Screen Descriptions

##### **3.1.2.1 Member Screen Descriptions**

| **#** | **Feature** | **Screen** | **Description** |
| --- | --- | --- | --- |
| 1 | App Bootstrapping | Splash | App startup screen that validates session state and performs initial routing. |
| 2 | Member Registration | Member Registration | Screen for capturing new member account credentials and details. |
| 3 | Member Registration | OTP Verification | Screen for validating member identity via phone or email OTP. |
| 4 | Member Registration | Registration Success | Screen displaying confirmation of successful member account creation. |
| 5 | Authentication | Member Login | Main authentication screen for member users to securely log in. |
| 6 | Authentication | Forgot Password | Screen to enter account/email and start the password recovery flow. |
| 7 | Authentication | Forgot Password OTP | OTP verification screen for the forgot-password flow. |
| 8 | Authentication | Reset Password | Screen for inputting and confirming a new secure password. |
| 9 | Authentication | Password Reset Success | Screen confirming that the password has been successfully updated. |
| 10 | Member Workspace | Member Home | Main dashboard providing quick access to emergency services and alerts. |
| 11 | Community Alerts | Community Alert | Screen displaying nearby active incidents and user-submitted reports. |
| 12 | Notification | Notification Tab | Centralized inbox for system alerts and message updates. |
| 13 | Emergency Response | Emergency Action | Entry point for initiating SOS and triggering snake identification. |
| 14 | Emergency Response | Snake Identification | Real-time AI camera interface for scanning and identifying snake species. |
| 15 | Emergency Response | Snake Selection by Location | Search interface for manually filtering snake species based on region. |
| 16 | Emergency Response | Snake Confirmation | Final confirmation screen verifying the identified snake species. |
| 17 | Clinical Assessment | Snake First Aid Guide | Step-by-step visual instruction guide for immediate medical response. |
| 18 | Clinical Assessment | Symptom Report | Form interface for capturing the patient's current physical symptoms. |
| 19 | Clinical Assessment | Severity Assessment | Automated system screen determining the clinical urgency level of the bite. |
| 20 | Emergency Tracking | Emergency Tracking | Live map interface tracking the inbound rescuer's geolocation. |
| 21 | Emergency Tracking | Member Incident Finished | Final mission summary detailing the resolved emergency incident. |
| 22 | Snake Catching | Snake Catching | Request interface for initiating non-emergency snake removal services. |
| 23 | Snake Catching | Snake Quantity Selection | Input screen for defining the estimated number of snakes to catch. |
| 24 | Snake Catching | Snake Report Detail | Form for providing additional situational context and attachments. |
| 25 | Snake Catching | Snake Catching Success | Confirmation screen validating the successful submission of the catch request. |
| 26 | Expert Consultation | Consultation Home | Central hub for navigating telemedicine and expert consultation services. |
| 27 | Expert Consultation | Expert List | Directory listing of verified medical and snake handling experts. |
| 28 | Expert Consultation | Expert Detail | Detailed profile showcasing expert qualifications, reviews, and availability. |
| 29 | Expert Consultation | Service Selection | Interface for selecting the appropriate tier or type of consultation. |
| 30 | Expert Consultation | Consultation Time Selection | Scheduling interface for booking future consultation appointments. |
| 31 | Expert Consultation | Consultation Documents | Upload screen for attaching media and medical notes prior to the call. |
| 32 | Expert Consultation | Payment Confirmation | Escrow payment gateway interface securing funds before the consultation. |
| 33 | Expert Consultation | Emergency Request Waiting | Queue interface while matching with an available on-call expert. |
| 34 | Expert Consultation | Video Waiting Room | Pre-call lobby verifying connection state before the live session. |
| 35 | Expert Consultation | Video Consultation | Live WebRTC video and audio interface for remote expert consultation. |
| 36 | Expert Consultation | Consultation Complete | Post-call summary providing medical prescriptions and expert notes. |
| 37 | Knowledge Base | Snake Library | Encyclopedic database of snake species, descriptions, and habitats. |
| 38 | Knowledge Base | Snake Detail | Specific informational profile detailing snake characteristics and risks. |
| 39 | Knowledge Base | Snake First Aid Guide | Comprehensive first-aid protocols mapped to specific snake species. |
| 40 | Knowledge Base | Blog List | Educational repository listing published safety and awareness articles. |
| 41 | Knowledge Base | Blog Detail | Article view screen for reading comprehensive educational content. |
| 42 | Member Profile | Profile Tab | User profile summary outlining personal information and status. |
| 43 | Member Profile | Edit Profile | Form interface for modifying user personal details and avatars. |
| 44 | Authentication | Role Selection | Screen for selecting the user role before sign-in or sign-up. |
| 45 | Community Alerts | Edit Community Report | Interface for modifying details of a previously submitted community alert. |
| 46 | Community Alerts | History Community | Historical log interface displaying the user's past community reports. |
| 47 | Community Alerts | Upload Community Report | Submission form for broadcasting a new community incident. |
| 48 | Emergency Tracking | Rescuer Arrived | Live status confirmation screen when the rescuer reaches the location. |
| 49 | Activity & History | Activity Tab | Overview dashboard summarizing the user's historical actions and requests. |
| 50 | Snake Catching | Snake Catching Detail | Detailed view of a past or completed snake catching mission. |
| 51 | Member Wallet | History Transaction | Comprehensive ledger of past fiat and wallet processing events. |
| 52 | Member Wallet | Transaction Detail | Specific receipt detailing a single financial transaction. |
| 53 | Member Wallet | Top-up | Payment gateway interface for adding funds to the user wallet. |
| 54 | Member Wallet | Withdrawal | Request interface for initiating a payout from the digital wallet. |
| 55 | Member Wallet | History Wallet | Overview interface tracking balance fluctuations and wallet history. |
| 56 | Expert Consultation | Scheduled Consultation | Log interface for viewing upcoming booked expert consultations. |
| 57 | Emergency Consultation | Emergency Consultation | Quick-access interface for initiating an immediate urgent consultation. |
| 58 | Member Profile | Settings | Application configuration interface for preferences and notification toggles. |
| 59 | Activity & History | Activity Detail | In-depth breakdown interface highlighting details of a specific past activity. |
| 60 | Activity & History | Activity History | Complete chronological list displaying all historical user events. |

###### Table 3.3.1.2.1 Member Screen Description

##### **3.1.2.2 Rescuer Screen Descriptions**

| **#** | **Feature** | **Screen** | **Description** |
| --- | --- | --- | --- |
| 1 | App Bootstrapping | Splash | App startup screen that validates session state and performs initial routing. |
| 2 | Authentication | Role Selection | Screen for selecting the user role before sign-in or sign-up. |
| 3 | Authentication | Rescuer Login | Login screen for rescuer users. |
| 4 | Authentication | Forgot Password | Screen to enter account/email and start the password recovery flow. |
| 5 | Authentication | Forgot Password OTP | OTP verification screen for the forgot-password flow. |
| 6 | Authentication | Reset Password | Screen for inputting and confirming a new secure password. |
| 7 | Authentication | Password Reset Success | Screen confirming that the password has been successfully updated. |
| 8 | Rescuer Workspace | Rescuer Home | Primary mission control dashboard providing daily statistics and current status. |
| 9 | Snake Catching | Available Jobs | List interface displaying open and active snake-catching requests nearby. |
| 10 | Snake Catching | Request Detail | Detailed view of a snake-catching request including photos and context. |
| 11 | Snake Catching | Accept Request | Confirmation dialog for committing to a dispatched rescue mission. |
| 12 | Snake Catching | En Route | Status screen tracking transit progress toward the target location. |
| 13 | Snake Catching | Tracking | Live monitoring interface maintaining system state during an active mission. |
| 14 | Snake Catching | Result Confirmation | Validation screen for confirming caught snake quantities and species. |
| 15 | Snake Catching | Mission Success - Snake Catching | Final summary screen concluding the completion of a snake catching job. |
| 16 | Emergency Response | Mission Detail - SOS | In-depth operational view for an active SOS medical emergency dispatch. |
| 17 | Emergency Response | Navigation Map | Real-time map interface mapping the optimal routing to the victim. |
| 18 | Emergency Response | On-scene Support | Interface detailing AI-recommended first-aid actions to perform on-scene. |
| 19 | Emergency Response | Find Hospital | Directory and routing interface for locating the nearest equipped hospital. |
| 20 | Emergency Response | Mission Completion | Check-out interface for finalizing the rescue operation and capturing evidence. |
| 21 | Emergency Response | Mission Success - Emergency | Final summary screen outlining the completed SOS emergency response. |
| 22 | Activity & History | Mission History | Chronological log displaying completed past rescue and catching operations. |
| 23 | Activity & History | History Detail | In-depth breakdown validating operational and financial specifics of a past mission. |
| 24 | Notification | Notification | Centralized paginated inbox handling system alerts and read/unread tracking. |
| 25 | Rescuer Profile | Profile Tab | Rescuer profile summary detailing performance metrics and ratings. |
| 26 | Rescuer Profile | Edit Profile | Form interface for modifying personal rescuer details and avatars. |
| 27 | Rescuer Profile | Settings | Application configuration interface for work modes and notification toggles. |
| 28 | Rescuer Reputation | Feedback | Interface for reviewing customer ratings and textual feedback. |
| 29 | Knowledge Base | Lessons | Educational hub providing modular training and procedural guides. |
| 30 | Knowledge Base | Lesson Detail | Content viewer screen for accessing specific training multimedia. |
| 31 | Knowledge Base | Snake Library | Encyclopedic database outlining snake species for field reference. |
| 32 | Knowledge Base | Snake Detail | Specific informational profile highlighting characteristics and handling risks. |
| 33 | Knowledge Base | First Aid Guide | Structured guide supplying exact first-aid procedures linked to species. |
| 34 | Schedule Management | Work Schedule Tab | View your assigned work schedule for the week. |

###### Table 3.3.1.2.2 Rescuer Screen Description

##### **3.1.2.3 Expert Screen Descriptions**

| **#** | **Feature** | **Screen** | **Description** |
| --- | --- | --- | --- |
| 1 | App Bootstrapping | Splash | App startup screen that validates session state and performs initial routing. |
| 2 | Authentication | Role Selection | Screen for selecting the user role before sign-in or sign-up. |
| 3 | Authentication | Expert Login | Login screen for expert users. |
| 4 | Authentication | Forgot Password | Screen to enter account/email and start the password recovery flow. |
| 5 | Authentication | Forgot Password OTP | OTP verification screen for the forgot-password flow. |
| 6 | Authentication | Reset Password | Screen to set a new password after OTP verification. |
| 7 | Authentication | Password Reset Success | Confirmation screen shown after password reset is completed. |
| 8 | Expert Registration | Expert Registration | Expert account registration screen. |
| 9 | Expert Registration | Expert Credentials | Screen to submit expert credentials and required professional information during registration. |
| 10 | Expert Registration | OTP Verification | OTP verification screen for expert account registration. |
| 11 | Expert Registration | Registration Success | Success screen shown after expert registration is submitted successfully. |
| 12 | Expert Registration | Registration Pending | Pending-review screen for expert registration approval. |
| 13 | Expert Workspace | Expert Home | Main expert home screen with primary tabs and dashboard overview. |
| 14 | Expert Profile | Profile Tab | Expert profile screen with personal info and related configuration entries. |
| 15 | Expert Profile | Settings | Expert settings screen. |
| 16 | Expert Profile | Edit Profile | Screen for editing expert profile details. |
| 17 | Expert Availability | Working Hours | Screen for configuring expert working schedule and available consultation slots. |
| 18 | Expert Wallet | Withdraw | Screen for withdrawing funds from the expert wallet. |
| 19 | Expert Reputation | Feedback | Screen for viewing user ratings and feedback for the expert. |
| 20 | Expert AI Review | AI Review Queue | Queue screen for pending AI recognition reviews. |
| 21 | Expert AI Review | AI Review Detail | Detail screen for reviewing and verifying/rejecting an AI recognition result. |
| 22 | Knowledge Base | Snake Library | Snake species library screen for expert users. |
| 23 | Knowledge Base | Snake Detail | screen to view details of the selected snake species in Expert Snake Library Screen. |
| 24 | Knowledge Base | Snake First Aid | First-aid guidance screen for snakebite handling (expert entry path). |
| 25 | Expert Content | Blog List | Screen listing the expert's blog posts. |
| 26 | Expert Content | Expert Blog Form | Screen for creating or editing an expert blog post. |
| 27 | Expert Consultation | Expert Consultation Detail | Screen showing detailed information for a selected expert consultation session (opened from Consultation List/History). |
| 28 | Expert Consultation | Expert Video Waiting | Expert waiting-room screen before joining the live video consultation. |
| 29 | Expert Consultation | Video Consultation | Live video consultation call screen. |
| 30 | Expert Consultation | Chat | A screen for the expert to message member during the consultation session. |
| 31 | Expert Consultation | Expert Consultation Complete | Expert completion screen shown after a consultation session ends. |
| 32 | Emergency Consultation | Expert Global Emergency Popup Listener | Global listener component/screen for emergency consultation request popups. |
| 33 | Emergency Consultation | Accept Emergency Request | State/screen for handling acceptance of an emergency request. |
| 34 | Emergency Consultation | Reject Emergency Request | State/screen for handling rejection of an emergency request. |
| 35 | Expert Consultation | Consultation List/History Tab | Consultation list/history section in the bottom navigation of Expert Home; selecting an item opens Expert Consultation Detail. |
| 36 | Notification | Notification | screen to view expert notifications. |
| 37 | Expert Income | Income Tab | screen to view expert earnings. |

###### Table 3.3.1.2.3 Expert Screen Description

##### **3.1.2.4 Admin and Operator Screen Descriptions**

| **#** | **Feature** | **Screen** | **Description** |
| --- | --- | --- | --- |
| 1 | Authentication | Web Login | Screen where user selects Management or Operator role before authentication. |
| 2 | Authentication | Management Login Form | Login form for Management account credentials in the same login page flow. |
| 3 | Authentication | Operator Login Form | Login form for Operator account credentials in the same login page flow. |
| 4 | Dashboard | Admin Dashboard | Main Management home screen with KPI cards, analytics charts (revenue/cases/profit/commission), period filter (day/month/year), recent incidents/requests, and Excel export. |
| 5 | Operator Operations | Operator Dashboard | Main and currently only implemented Operator route; includes live map, incident/request info panels, shift schedule panel, pending alerts, incident detail modal, request detail modal, and dispatch/assignment actions. |
| 6 | Workforce Management | Workshifts Management | Shift management screen with shift template list, calendar assignments, today's assignments, rescuer detail panel, and single/bulk assignment flows (including check-in/check-out). |
| 7 | User Management | Users Management | User management screen with stats cards, user growth chart, role/status filters, user list, and right-side user detail panel with account actions. |
| 8 | Incident Management | Incidents Management | Incident management screen with 2 internal tabs (Incidents and Missions), each with list view and corresponding detail modal flow. |
| 9 | Incident Management | Snake Catching Management | Snake catching request management screen with keyword search, essential list fields, action column with detail button, detail modal fed by request-detail endpoint, and structured sections for requester, location, assigned rescuer, species details, and media. |
| 10 | Consultation Management | Consultations Management | Consultation session management screen with type/status filters, paginated list, action column with detail button, and full detail modal for timeline, participants, references, and pricing context. |
| 11 | Snake Data Management | Snakes Management | Snake species management screen with species list, create/edit upsert modal, prevalence map section, and linked antivenom references. |
| 12 | Medical Inventory | Antivenoms Management | Antivenom management screen with list view, selected-detail panel, and upsert modal for create/edit. |
| 13 | Facility Management | Treatment Facilities Management | Treatment facility management screen with list view, selected-detail panel, and upsert modal including antivenom association settings. |
| 14 | Finance Management | Transactions & Withdrawals Management | Finance operations screen with 2 internal tabs (Transactions and Withdrawals), transaction filters (user/type), paginated list, and transaction detail modal. |
| 15 | System Configuration | Settings Management | Dynamic settings management screen with searchable list, detail panel, and create/edit modal supporting value types. |
| 16 | Report Media Management | Report Media Management | Multi-source report media management screen (Community Report, Snakebite Incident, Rescue Mission, Snake Catching Request/Mission) with status/reference/date filters, paginated media list, and detail modal for full recognition metadata/review state. |
| 17 | Content Management | Lessons Management | Lesson CMS screen with category/publish filters, list + detail panel layout, lesson upsert modal, and lesson content preview section. |
| 18 | Content Management | Blogs Management | Blog CMS screen with status filter tabs, list + detail panel layout, blog upsert modal, content preview, and moderation flow (approve/reject with reason). |

###### Table 3.3.1.2.4 Admin and Operator Screen Description

#### 3.1.3 Screen Authorization

| **Screen** | **Member** | **Rescuer** | **Expert** | **Admin** | **Operator** |
| --- | --- | --- | --- | --- | --- |
| Splash | X | X | X |  |  |
| Role Selection | X | X | X |  |  |
| Member Login | X |  |  |  |  |
| Rescuer Login |  | X |  |  |  |
| Expert Login |  |  | X |  |  |
| Forgot Password | X | X | X |  |  |
| Forgot Password OTP | X | X | X |  |  |
| Reset Password | X | X | X |  |  |
| Password Reset Success | X | X | X |  |  |
| Member Registration | X |  |  |  |  |
| Expert Registration |  |  | X |  |  |
| Expert Credentials |  |  | X |  |  |
| OTP Verification | X | X | X |  |  |
| Registration Success | X | X | X |  |  |
| Registration Pending |  |  | X |  |  |
| Member Home | X |  |  |  |  |
| Rescuer Home |  | X |  |  |  |
| Expert Home |  |  | X |  |  |
| Notification Tab | X | X | X |  |  |
| Work Schedule Tab |  | X |  |  |  |
| Community Alert | X |  |  |  |  |
| Emergency Action | X |  |  |  |  |
| Snake Identification | X |  |  |  |  |
| Snake Selection by Location | X |  |  |  |  |
| Snake Confirmation | X |  |  |  |  |
| First Aid Steps | X |  |  |  |  |
| Symptom Report | X |  |  |  |  |
| Severity Assessment | X |  |  |  |  |
| Emergency Tracking | X |  |  |  |  |
| Member Incident Finished | X |  |  |  |  |
| Snake Catching | X |  |  |  |  |
| Snake Quantity Selection | X |  |  |  |  |
| Snake Report Detail | X |  |  |  |  |
| Snake Catching Success | X |  |  |  |  |
| Consultation Home | X |  |  |  |  |
| Expert List | X |  |  |  |  |
| Expert Detail | X |  |  |  |  |
| Service Selection | X |  |  |  |  |
| Consultation Time Selection | X |  |  |  |  |
| Consultation Documents | X |  |  |  |  |
| Payment Confirmation | X |  |  |  |  |
| Emergency Request Waiting | X |  |  |  |  |
| Video Waiting Room | X |  |  |  |  |
| Video Consultation | X |  | X |  |  |
| Consultation Complete | X |  |  |  |  |
| Snake Library | X | X | X |  |  |
| Snake Detail | X | X | X |  |  |
| Snake First Aid Guide | X | X | X |  |  |
| Blog List | X |  | X |  |  |
| Blog Detail | X |  |  |  |  |
| Expert Blog Form |  |  | X |  |  |
| Profile Tab | X | X | X |  |  |
| Edit Profile | X | X | X |  |  |
| Edit Community Report | X |  |  |  |  |
| History Community | X |  |  |  |  |
| Upload Community Report | X |  |  |  |  |
| Rescuer Arrived | X |  |  |  |  |
| Activity Tab | X |  |  |  |  |
| Snake Catching Detail | X |  |  |  |  |
| History Transaction | X |  |  |  |  |
| Transaction Detail | X |  |  |  |  |
| Top-up | X |  |  |  |  |
| Withdrawal | X |  | X |  |  |
| History Wallet | X |  |  |  |  |
| Scheduled Consultation | X |  |  |  |  |
| Emergency Consultation | X |  |  |  |  |
| Settings | X | X | X |  |  |
| Activity Detail | X |  |  |  |  |
| Activity History | X |  |  |  |  |
| Available Jobs |  | X |  |  |  |
| Request Detail |  | X |  |  |  |
| Accept Request |  | X |  |  |  |
| En Route |  | X |  |  |  |
| Tracking |  | X |  |  |  |
| Result Confirmation |  | X |  |  |  |
| Mission Success - Snake Catching |  | X |  |  |  |
| Mission Detail - SOS |  | X |  |  |  |
| Navigation Map |  | X |  |  |  |
| On-scene Support |  | X |  |  |  |
| Find Hospital |  | X |  |  |  |
| Mission Completion |  | X |  |  |  |
| Mission Success - Emergency |  | X |  |  |  |
| Mission History |  | X |  |  |  |
| History Detail |  | X |  |  |  |
| Feedback |  | X | X |  |  |
| Lessons |  | X |  |  |  |
| Lesson Detail |  | X |  |  |  |
| Working Hours |  |  | X |  |  |
| AI Review Queue |  |  | X |  |  |
| AI Review Detail |  |  | X |  |  |
| Expert Consultation Detail |  |  | X |  |  |
| Expert Video Waiting |  |  | X |  |  |
| Expert Consultation Complete |  |  | X |  |  |
| Expert Global Emergency Popup Listener |  |  | X |  |  |
| Accept Emergency Request |  |  | X |  |  |
| Reject Emergency Request |  |  | X |  |  |
| Consultation List/History Tab |  |  | X |  |  |
| Income Tab |  |  | X |  |  |
| Web Login |  |  |  | X | X |
| Admin Login Form |  |  |  | X |  |
| Operator Login Form |  |  |  |  | X |
| Admin Dashboard |  |  |  | X |  |
| Operator Dashboard |  |  |  |  | X |
| Workshifts Management |  |  |  | X |  |
| Users Management |  |  |  | X |  |
| Incidents Management |  |  |  | X |  |
| Snake Catching Management |  |  |  | X |  |
| Consultations Management |  |  |  | X |  |
| Snakes Management |  |  |  | X |  |
| Antivenoms Management |  |  |  | X |  |
| Treatment Facilities Management |  |  |  | X |  |
| Transactions & Withdrawals Management |  |  |  | X |  |
| Management Withdrawals |  |  |  | X |  |
| Settings Management |  |  |  | X |  |
| Report Media Management |  |  |  | X |  |
| Lessons Management |  |  |  | X |  |
| Blogs Management |  |  |  | X |  |

###### Table 3.3.1.3.1 Screen Authorization

#### 3.1.4 Non-Screen Functions

| **#** | **Feature** | **System Function** | **Description** |
| --- | --- | --- | --- |
| 1 | Authentication and  Authorization | JWT Token  Generation and  Verification | Supabase issued token (JWT) and parsing,  verification process to identify the user's role and permission |

###### Table 3.3.1.4.1 Non-Screen Functions

####

####

####

####

#### 3.1.5 Entity Relationship Diagram![](data:image/png;base64...)

###### Figure 3.3.1.5.1 Entity Relationship Diagram

**Entities Description**

| **#** | **Entity** | **Description** |
| --- | --- | --- |
| 1 | Account | Represents the core authentication and identity record for every user in the system. It stores login credentials and basic identity information, and serves as the root entity linked to all role-specific profiles. |
| 2 | MemberProfile | Represents a general user profile associated with an account. Members can report snake-related incidents, request snake-catching services, and participate in consultations. |
| 3 | RescuerProfile | Represents an operational profile responsible for executing rescue and snake-catching missions. It includes availability status, service capability, and performance-related data. |
| 4 | ExpertProfile | Represents a specialist profile linked to an account, including professional credentials, areas of expertise, and consultation-related configurations such as pricing and availability. |
| 5 | SnakebiteIncident | Represents an emergency case reported after a snakebite event. It captures critical information such as location, time, victim condition, and symptoms, and acts as the trigger for the emergency response workflow. |
| 6 | RescueRequest | Represents a dispatch request created to assign a rescuer to a snakebite incident. It functions as a coordination layer between incident reporting and mission execution. |
| 7 | RescueMission | Represents the execution of a rescue operation assigned to a rescuer. It tracks the lifecycle of the mission, including assignment, progress, and completion outcome. |
| 8 | SnakeCatchingRequest | Represents a service request created by a member for snake-catching assistance, including location and request details. |
| 9 | CatchingRequestDetail | Stores detailed information about requested snake species and quantities associated with a snake-catching request. |
| 10 | SnakeCatchingMission | Represents a mission assigned to a rescuer to handle a snake-catching request, including execution tracking and outcome. |
| 11 | CatchingMissionDetail | Records the actual snake species captured and quantities during the execution of a snake-catching mission. |
| 12 | CatchingEnvironment | Represents environmental factors (e.g., urban, rural) used to determine pricing or difficulty levels for snake-catching operations. |
| 13 | Consultation | Represents a real-time interaction session between two accounts (e.g., member and expert) for advice, guidance, or support. |
| 14 | ConsultationBooking | Represents the scheduling and payment record for a consultation session, including booking status and lifecycle. |
| 15 | ExpertTimeSlot | Defines available time slots for experts, which can be booked by users for consultation sessions. |
| 16 | ChatMessage | Represents a message exchanged between participants within a consultation session. |
| 17 | SnakeSpecies | Represents the central knowledge entity for snake taxonomy, including identification characteristics, risk level, and related medical information. |
| 18 | SnakeSpeciesName | Stores alternative or localized names associated with a snake species. |
| 19 | VenomType | Represents categories of snake venom, including their characteristics and severity levels. |
| 20 | Antivenom | Represents antivenom products used for treating snakebite cases. |
| 21 | FirstAidGuideline | Represents structured first-aid instructions used to guide users in handling snakebite situations. |
| 22 | SymptomConfig | Defines configurable rules and scoring logic for evaluating symptoms and assessing snakebite severity. |
| 23 | TreatmentFacility | Represents hospitals or medical facilities capable of providing treatment for snakebite cases. |
| 24 | GeographicRegion | Represents a geographic area defined by spatial boundaries, used for location-based filtering and mapping. |
| 25 | AIModel | Represents metadata of deployed AI models used for snake recognition, including versioning and configuration. |
| 26 | SnakeAIRecognitionResult | Represents the result of snake identification generated by an AI model based on uploaded media. |
| 27 | ReportMedia | Represents media files (images, videos) attached to various entities such as incidents, missions, and reports using a polymorphic association. |
| 28 | LibraryMedia | Represents curated media content stored in the system, optionally associated with snake species for educational purposes. |
| 29 | CommunityReport | Represents user-generated reports about snake sightings or related observations, including optional media and location data. |
| 30 | Blog | Represents educational or informational articles created by platform users or administrators. |
| 31 | Lesson | Represents structured learning content used for educational features within the system. |
| 32 | UserFeedback | Represents ratings and feedback exchanged between users after service interactions. |
| 33 | AppNotification | Represents in-app notifications sent to users, including system messages and event updates. |
| 34 | Wallet | Represents a user’s wallet used to store balance and manage financial transactions within the platform. |
| 35 | WalletWithdraw | Represents a withdrawal request from a user’s wallet, including approval and processing lifecycle. |
| 36 | Transaction | Represents a financial transaction record associated with payments, services, or wallet operations. |
| 37 | PaymentCard | Represents stored payment card information used for processing transactions. |
| 38 | WorkShift | Represents predefined work shift templates used for scheduling rescuers. |
| 39 | SystemSetting | Represents configurable key-value settings used to control system behavior at runtime. |
| 40 | Specialization | Represents categories of expertise used to classify expert knowledge domains. |
| 41 | ExpertCertificate | Represents certification records validating the qualifications of experts. |

###### Table 3.3.1.5.2 Entities Description

### 3.2 Authentication Feature

#### 3.2.1 Mobile Login

* **Function trigger:** User opens the mobile app and selects a role. Navigation path: Splash -> Role Selection -> Login.
* **Function description:**
  + **Actor:** Member, Rescuer, Expert.
  + **Purpose:** Authenticate user account and enter the role-specific portal.
  + **Interface:** Role selector and role-specific login form with forgotten password entry.
  + **Data processing:** Submit credentials, validate account status and selected role, create session/token, redirect to corresponding home/dashboard.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.2.1.1 Splash Screen

![](data:image/png;base64...)

###### Figure 3.3.2.1.2 Expert Login Screen

* **Function details:**
  + **Data:** Input: email/username, password. Output: authenticated session, profile context.
  + **Validation:** Required credential fields must be provided; Account must match selected role.
  + **Business rules:** Login success must route users to the correct role-specific module.
  + **Normal cases:** Successful login redirects to the corresponding role home/dashboard.
  + **Abnormal cases:** Invalid credentials; Account inactive/locked; API timeout or server error.

####

####

####

#### 3.2.2 Forgot Password

* **Function trigger:** User taps forgot password from Login. Navigation path: Login -> Forgot Password -> Forgot Password OTP -> Reset Password -> Password Reset Success.
* **Function description:**
  + **Actor:** Member, Rescuer, Expert.
  + **Purpose:** Recover account access through OTP verification and password reset.
  + **Interface:** Request form, OTP input, reset password form, completion screen.
  + **Data processing:** Request OTP, validate OTP, update password, confirm completion.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.2.2.1 Forgot Password Screen

* **Function details:**
  + **Data:** Input: account/email, OTP code, new password and confirmation. Output: password reset result.
  + **Validation:** OTP must be valid and unexpired; New password must meet security policy; Password confirmation must match.
  + **Business rules:** Passwords can only be changed after successful OTP verification.
  + **Normal cases:** User resets password and can return to login.
  + **Abnormal cases:** Invalid/expired OTP; Weak password; Retry limit exceeded.

#### 3.2.3 Member Registration

* **Function trigger:** App launches unauthenticated. Navigation path: Splash -> Role Selection -> Member Registration -> OTP Verification -> Registration Success
* **Function description:**
  + **Actor:** Expert / Guest.
  + **Purpose:** Securely onboard new members, verify identity, and manage authenticated session lifecycles.
  + **Interface:** Role selection cards, data capture forms, OTP keypad, and standardized identity validation views.
  + **Data processing:** Issue registration payloads to Auth service, dispatch and verify Email OTP, return session tokens.
* **Screen layout:**

*![](data:image/png;base64...)*

###### Figure 3.3.2.3.1 Member Registration Screen

*![](data:image/png;base64...)*

###### Figure 3.3.2.3.2 Registration Success Screen

* **Function details:**
  + **Data:** PII credentials (phone, email, name, etc...), validation tokens, JWT session objects.
  + **Validation:** Enforce strong password complexity; Ensure unique identity (no duplicate emails/phones); OTP strict expiration.
  + **Business rules:** Unverified accounts cannot access core emergency or consultation features; Logins demand exact credential matches.
  + **Normal cases:** User registers, validates OTP, and seamlessly drops into Member Workspace.
  + **Abnormal cases:** Identity collision triggers "Account Exists" warning; Exhausted OTP attempts halt registration temporarily.

####

####

#### 3.2.4 Expert Registration

* **Function trigger:** User chooses to register as expert from role selection/login area. Navigation path: Role Selection -> Expert Registration -> OTP Verification -> Registration Success -> Expert Credentials.
* **Function description:**
  + **Actor:** Expert / Guest.
  + **Purpose:** Submit expert registration and required credentials for approval workflow.
  + **Interface:** Registration form, credentials screen, OTP verification, pending status, success notification.
  + **Data processing:** Create registration, upload credentials, verify contact by OTP, mark request as pending review, activate on approval.
* **Screen layout:**

**![](data:image/png;base64...)**

###### Figure 3.3.2.4.1 Expert Registration Screen

**![](data:image/png;base64...)**

###### Figure 3.3.2.4.2 Expert Registration Success Screen

**![](data:image/png;base64...)**

###### Figure 3.3.2.4.3 Expert Credentials Screen

* **Function details:**
  + **Data:** Input: personal info, account credentials, professional credentials, OTP. Output: registration request status and account activation outcome.
  + **Validation:** Required registration and credential fields must be complete; OTP must be valid for registration confirmation.
  + **Business rules:** Registration must pass pending review before full portal access.
  + **Normal cases:** Applicant submits profile and enters pending/approved flow.
  + **Abnormal cases:** Duplicate account; Invalid credential payload; OTP verification failure

#### 3.2.5 Rescuer Registration

* **Function trigger:** Web log as admin account. Navigation path: Login Web-> Admin dashboard -> User Management -> Register Rescuer.
* **Function description:**
  + **Actor:** Admin.
  + **Purpose:** create verified rescuer for account.
  + **Interface:** Modal for rescuer.
  + **Data processing:** Issue registration payloads to Auth service, dispatch and verify Email, password, phone Number.
* **Screen layout:**

*![](data:image/png;base64...)*

###### Figure 3.3.2.5.1 Rescuer Registration Modal

* **Function details:**
  + **Data:** PII credentials (phone, email, name, etc...), validation tokens, JWT session objects.
  + **Validation:** Enforce strong password complexity; Ensure unique identity (no duplicate emails/phones).
  + **Business rules:** only Admin can create a rescuer account,=.
  + **Normal cases:** Admin registers, input name, email, password, type of rescue , and rescuer can log into Rescuer Workspace.

#### Abnormal cases: Identity collision triggers "Account Exists" warning.

#### 3.2.6 Web Login

* **Function trigger:**
  + The user opens the Web Login page.
  + Navigation path: Web Login -> role selection -> Admin Login Form or Operator Login Form.
* **Function description:**
  + **Actors:**Admin, Operator.
  + **Purpose:** Authenticate users by role before entering the operation/admin portal.
  + **Interface:** Role selector and role-specific login form.
  + **Data processing:** Submit credentials, validate role compatibility, create session/token, redirect to the corresponding dashboard.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.2.6.1 Web Login Screen

![](data:image/png;base64...)

###### Figure 3.3.2.6.2 Admin Login Form Screen

* **Function details:**
  + **Data:**
    - **Input:** email/username, password, selected role.
    - **Output:** authenticated session and role-based user profile.
  + **Validation:**
    - All required fields must be provided.
    - The selected role must match the account role.
  + **Business rules:**
    - Admins can access Admin Management only.
    - Operators can access Operator Dashboard only.
  + **Normal cases:** Successful login redirects to the correct role dashboard.
  + **Abnormal cases:** Invalid credentials; Role mismatch; API error or timeout.

### 3.3 Member Portal Feature

#### 3.3.1 Member Workspace & Notifications

* **Function trigger:** Authenticated user enters the app. Navigation path: Member Login -> Member Home <-> Notification Tab.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Serve as the primary routing hub, surfacing critical emergency CTAs and personal alerts.
  + **Interface:** Dominant SOS action hero button, service navigation grid, and system alert inbox.
  + **Data processing:** Fetch active user state, aggregate unread notification counts, identify any ongoing active emergency operations.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.1.1 Member Home Screen

![](data:image/png;base64...)

###### Figure 3.3.3.1.2 Notification Tab Screen

* **Function details:**
  + **Data:** User contextual profile, system notification payload array, active mission state flags.
  + **Validation:** Valid member session.
  + **Business rules:** If the member has an active emergency dispatch, the Home screen must forcefully display an immediate "Return to Tracking" persistent banner.
  + **Normal cases:** Workspace renders fully; User taps Notification Tab to review unread system updates.
  + **Abnormal cases:** Network failure gracefully loads offline cache with a disabled SOS warning banner.

####

#### 3.3.2 Community Alerts

* **Function trigger:** User investigates local environment safety. Navigation path: Member Home -> Community Alert -> (Upload Community Report) or (Edit Community Report / History Community).
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Enable crowdsourced hazard reporting and situational awareness of nearby snake sightings.
  + **Interface:** Geospatial map/list of incidents, media upload forms, and historical logs of personal reports.
  + **Data processing:** Query geofenced hazard datasets, process multipart form uploads (images + location), log reporting history..
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.2.1 Community Alert Screen

![](data:image/png;base64...)

###### Figure 3.3.3.2.2 Upload Community Report Screen

![](data:image/png;base64...)

###### Figure 3.3.3.2.3 History Community Screen

![](data:image/png;base64...)

###### Figure 3.3.3.2.4 Edit Community Report Screen

* **Function details:**
  + **Data:** Incident coordinates, sighting descriptions, image media, timestamp metadata.
  + **Validation:** Location coordinates are mandatory; At least one visual evidence attachment must be provided.
  + **Business rules:** User-submitted reports append a "Community Verified" metadata flag; Users can only edit their own active reports.
  + **Normal cases:** Member spots a hazard, maps the coordinates, uploads a photo, and the alert broadcasts to nearby users.
  + **Abnormal cases:** GPS permission denied blocks report creation; Media upload timeouts generate robust retry prompts.

####

#### 3.3.3 Emergency SOS & Snake Identification

* **Function trigger:** Member triggers critical emergency workflow.
* **Navigation path**:
  + Primary Path: Member Home -> Emergency Action -> AI Snake Identification -> Snake Confirmation.
  + Fallback Path: Member Home -> Emergency Action -> Snake Selection by Location -> Snake Confirmation.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Provide a rapid, high-stress interface to initiate SOS protocols and utilize AI for immediate threat identification, with a location-based manual fallback to ensure zero-failure in identification.
  + **Interface:** Distraction-free camera scanner, ML bounding-box overlay, geolocation-filtered species list, and definitive confirmation modals.
  + **Data processing:** Stream camera frames to Edge/Cloud ML models for real-time inference. If confidence thresholds are not met, the system executes geospatial narrowing to fetch a localized species database for manual selection.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.3.1 Emergency Action Screen

![](data:image/png;base64...)

###### Figure 3.3.3.3.2 Emergency Tracking Screen

![](data:image/png;base64...)

###### Figure 3.3.3.3.3 Snake Identification Screen

![](data:image/png;base64...)

###### Figure 3.3.3.3.4 Snake Confirmation Screen

![](data:image/png;base64...)

###### Figure 3.3.3.3.5 Snake Selection by Location Screen

![](data:image/png;base64...)

###### Figure 3.3.3.3.6 Confirm Snake Selection by Location Modal

* **Function details:**
  + **Data:** Real-time visual frames, inferred species ID arrays, confidence thresholds, GPS coordinates, and regional species metadata.
  + **Validation:** Device camera and high-accuracy location permissions are strictly required; biometric/PIN bypass is enabled for speed.
  + **Business rules:** Execution speed is paramount; AI inference must return within set latency bounds or auto-trigger the location-based manual fallback flow.
  + **Normal cases:** AI detects species with high confidence -> User confirms -> System progresses to Clinical Assessment.
  + **Abnormal cases:** Pitch-black image or blurry motion fails AI thresholds -> Instantly redirects user to manual "Snake Selection by Location".

#### 3.3.4 Clinical Assessment & First Aid

* **Function trigger:** SOS species are confirmed or declared unknown. Navigation path: Snake Confirmation -> First Aid Steps -> Symptom Report -> Severity Assessment.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Administer immediate life-saving directives and autonomously triage the victim's clinical severity to inform dispatch systems.
  + **Interface:** High-visibility procedural steps, symptom ticking checklists, and urgent severity determination outcome screens.
  + **Data processing:** Map identified species to specific clinical protocols, process selected symptoms against severity matrix algorithms.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.4.1 First Aid Steps Screen

![](data:image/png;base64...)

###### Figure 3.3.3.4.2 Symptom Report Screen

![](data:image/png;base64...)

###### Figure 3.3.3.4.3 Severity Assessment Screen

* **Function details:**
  + **Data:** Snake species ID, Boolean symptom array, computed triage level (e.g., Low, High, Critical).
  + **Validation:** At least one symptom state (including "No symptoms") must be explicitly selected to advance.
  + **Business rules:** App never provides definitive medical diagnoses, only algorithmic triage to rank dispatch priorities; Unknown species default to High/Critical precaution levels.
  + **Normal cases:** Member reads protocol -> checks symptoms -> system determines "Critical" -> directly initiates Rescuer dispatch sequence.
  + **Abnormal cases:** User abandons flow mid-way -> System holds the SOS state active and prompts resumption upon next app launch.

#### 3.3.5 Emergency Tracking & Incident Billing

* **Function trigger:** SOS is actively dispatched to a rescuer. Navigation path: Severity Assessment -> Emergency Tracking -> Rescuer Arrived -> Member Incident Finished -> Payment Interface.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Provide psychological relief and operational visibility by tracking the inbound emergency responder in real-time, and settle the financial invoice once the threat is resolved.
  + **Interface:** Live map rendering dynamic polylines, ETA countdowns, responder profile snippets, final incident resolution summaries, and billing checkout interface.
  + **Data processing:** Consume incoming WebSocket/SignalR geolocation points, calculate route recalculations, sync final state closure, and interface with SnakeAidPay Wallet or PayOS gateway for bill settlement.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.5.1 Emergency Tracking Screen - Status Mission “En-route”

![](data:image/png;base64...)

###### Figure 3.3.3.5.2 Rescuer Arrived Screen

![](data:image/png;base64...)

###### Figure 3.3.3.5.3 Member Incident Finished Screen

![](data:image/png;base64...)

###### Figure 3.3.3.5.4 Member Selection Payment Modal

![](data:image/png;base64...)

###### Figure 3.3.3.5.5 Member Payment Successful Modal

* **Function details:**
  + **Data:** Rescuer live coordinates, updated ETA metrics, discrete mission states (Assigned, En Route, Arrived, Resolved), billing invoice totals.
  + **Validation:** Ensures rescuer maintains an active transmit heartbeat. Payment requires sufficient wallet balance.
  + **Business rules:**
    - Members cannot abort the mission once the rescuer transitions to "Arrived" state.
    - Execute-First, Pay-Later Priority: SOS operations bypass upfront payments to prioritize life-safety. Payment is mandated only after the rescuer marks the incident as "Hoàn thành cứu hộ",
  + **Normal cases:** escuer dot approaches on map -> State flips to Arrived -> Operation concludes -> System generates billing summary -> Member pays via SnakeAidPay Wallet or directly via PayOS.
  + **Abnormal cases:**
    - Rescuer goes offline -> UI shows "Signal Lost" while system attempts re-routing or re-assignment in background.
    - Payment Failed/Canceled -> Incident cannot be closed until Member successfully completes the payment.

#### 3.3.6 Snake Catching Request & Initial Payment

* **Function trigger:** Member opts for non-medical snake removal. Navigation path: Member Home -> Snake Catching -> Snake Quantity Selection -> Snake Report Detail -> Snake Catching Success -> Initial Payment (Travel Fee).
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Orchestrate the creation of a professional, non-urgent snake catching request and process the upfront travel fee.
  + **Interface:** Address confirmation map, quantity counter, environmental context forms, checkout interface (phase 1), and success confirmation.
  + **Data processing:** Geocode address endpoints, construct dispatch payloads, query availability of non-emergency responders, and process the initial payment via SnakeAidPay Wallet or PayOS.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.6.1 Snake Quantity Selection Screen

![](data:image/png;base64...)

###### Figure 3.3.3.6.2 Snake Report Detail Screen

![](data:image/png;base64...)

###### Figure 3.3.3.6.3 Snake Report Detail Screen (AI result)

![](data:image/png;base64...)

###### Figure 3.3.3.6.4 Snake Catching Success Screen

![](data:image/png;base64...)

###### Figure 3.3.3.6.5 Member Selection Payment Modal (Travel Fee)

![](data:image/png;base64...)

###### Figure 3.3.3.6.6 Member Payment Successful Modal (Travel Fee)

![](data:image/png;base64...)

###### Figure 3.3.3.6.7 Snake Catching Detail Screen

* **Function details:**
  + **Data:** Geocoordinates, quantity integer, environmental text description, attached situational photographs, Phase 1 invoice (Travel fee).
  + **Validation:** Provided address must fall within the platform's operational service polygons. Payment requires sufficient wallet balance or active PayOS transaction success.
  + **Business rules:**
    - Snake Catching requests explicitly sit at a lower dispatch priority compared to SOS Medical workflows.
    - Phase 1 Payment: Member must pay the "Travel Fee" (Phí di chuyển) before the system dispatches a snake catcher.
  + **Normal cases:** Member defines parameters, submits form -> Pays Travel Fee -> System secures a catcher -> Shows success confirmation dispatch.
  + **Abnormal cases:**
    - No active responders available -> UI declines the request.
    - Phase 1 Payment Fails -> Request is aborted/not dispatched.

#### 3.3.7 Snake Catching Tracking & Final Payment

* **Function trigger:** Snake catching request is accepted by a rescuer. Navigation path: Activity Tab -> Activity Detail -> Mission Execution -> Final Payment (Service Fee).
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Allow members to track the ongoing snake catching mission and finalize the remaining service fee once the rescuer completes the job.
  + **Interface:** status updates, rescuer profile snippets, mission completion summary, and phase 2 checkout interface.
  + **Data processing:** Consume incoming location/status updates, sync final mission closure, and process the final payment phase via SnakeAidPay Wallet or PayOS.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.7.1 Snake Catching Detail Screen (when status “En-Route”)

![](data:image/png;base64...)

###### Figure 3.3.3.7.2 Snake Catching Detail Screen (when status “Arrived”)

![](data:image/png;base64...)

###### Figure 3.3.3.7.3 Member Selection Payment (Catching Payment)

![](data:image/png;base64...)

###### Figure 3.3.3.7.4 Snake Catching Detail Screen (when status “Completed”)

* **Function details:**
  + **Data:** Rescuer live coordinates, discrete mission states (En Route, Arrived, Resolved), Phase 2 invoice (Remaining service fee).
  + **Validation:** Payment requires sufficient wallet balance or active PayOS transaction success.
  + **Business rules:**
    - Phase 2 Payment: Member pays the "Remaining Balance" (Phí dịch vụ còn lại) once the catcher marks the mission as successfully resolved.
  + **Normal cases:** Catcher arrives -> Completes the job -> System generates final bill -> Member pays Remaining Balance via Wallet/PayOS.
  + **Abnormal cases:**
    - Phase 2 Payment Fails -> Incident remains in "Pending Final Settlement" state until the user successfully completes the second payment.

#### 3.3.8 Scheduled Expert Consultation

* **Function trigger:** Member requires scheduled professional clinical or zoological advice. Navigation path: Member Home -> Consultation Home -> Expert List -> Expert Detail -> Service Selection -> Consultation Time Selection -> Consultation Documents -> Payment Confirmation.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Schedule future synchronous telemedicine and expert advisory sessions.
  + **Interface:** Filterable expert directories, calendar pickers, medical document upload forms, and checkout gateways.
  + **Data processing:** Execute calendar scheduling logic, process escrow payment transactions.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.8.1 Consultation Home Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.2 Expert List Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.3 Expert Detail Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.4 Service Selection Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.5 Scheduled Consultation Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.6 Consultation Documents Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.7 Payment Confirmation Screen

![](data:image/png;base64...)

###### Figure 3.3.3.8.8 Consultation Home Screen (when it have the booking)

* **Function details:**
  + **Data:** Selected expert ID, ISO8601 timeslots, multipart clinical documents, payment intent tokens.
  + **Validation:** Scheduled timeslots must strictly avoid overlap; Escrow payment capture must perfectly succeed before session locks.
  + **Business rules:** Scheduled sessions commit funds into escrow pending successful timeline execution and session completion.
  + **Normal cases:** Member schedules doc -> pays -> receives scheduled appointment confirmation.
  + **Abnormal cases:** Scheduling conflicts return block errors; Payment gateway rejects.

####

#### 3.3.9 Instant Expert Consultation

* **Function trigger:** Member requires immediate professional advice without waiting. Navigation path: Member Home -> Consultation Home -> Expert List -> Expert Detail -> Service Selection -> Consultation Documents -> Payment Confirmation -> Emergency Request Waiting.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Immediately request an ad-hoc consultation with an available on-call expert and enter the priority queue.
  + **Interface:** Filterable active directories, instant-connect medical document upload forms, checkout gateways, and queue waiting interface.
  + **Data processing:** Query active status (On-Call) logic, process immediate escrow payment, assign expert.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.9.1 Expert List Screen (Have Expert Online)

![](data:image/png;base64...)

###### Figure 3.3.3.9.2 Expert Detail Screen (Expert Online)

![](data:image/png;base64...)

###### Figure 3.3.3.9.3 Service Selection Screen (Expert Online)

![](data:image/png;base64...)

###### Figure 3.3.3.9.4 Payment Confirmation Screen

![](data:image/png;base64...)

###### Figure 3.3.3.9.5 Emergency Request Waiting Modal

* **Function details:**
  + **Data:** Targeted active expert ID, multipart clinical payloads, payment intent tokens.
  + **Validation:** Selected experts must be flagged 'Active/On-Call'; Payment processing must be prioritized for instant clearance.
  + **Business rules:** Bypasses standard scheduling grids; Locks current availability immediately upon successful escrow commit.
  + **Normal cases:** Member selects on-call expert -> pays -> instantly routes to Emergency Request Waiting.
  + **Abnormal cases:** Selected expert drops offline just before payment clears (system triggers refund or re-routes).

#### 3.4.10 Video Consultation & Completion

* **Function trigger:** Time arrives for a scheduled appointment OR an instant consultation request is accepted. Navigation path: Video Waiting Room -> Video Consultation -> Chat Consultation -> Consultation Complete.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Facilitate the actual synchronous telemedicine video session.
  + **Interface:** Pre-call lobby, live WebRTC video and audio interfaces.
  + **Data processing:** Instantiate WebRTC signaling and media streams, process session completion to release escrow funds.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.10.1 Video Waiting Room Screen

![](data:image/png;base64...)

###### Figure 3.3.3.10.2 Video Consultation Screen

![](data:image/png;base64...)

###### Figure 3.3.3.10.3 Box Chat Modal

![](data:image/png;base64...)

###### Figure 3.3.3.10.4 Consultation Complete Screen

* **Function details:**
  + **Data:** RTC connection descriptors, final medical prescription data.
  + **Validation:** Device camera/mic permissions required. Stable network connection.
  + **Business rules:** Consultation is officially marked complete only after the expert ends the session and provides the summary, triggering the release of escrow funds.
  + **Normal cases:** Member enters waiting room -> connects with expert -> finishes call -> views completion summary.
  + **Abnormal cases:** WebRTC ICE failure drops video -> UI gracefully downgrades to audio-only or text chat; ICE server fails connection completely.

####

#### 3.3.11 Snake Library

* **Function trigger:** Member accesses educational resources about snakes. Navigation path: Member Home -> Snake Library -> Snake Detail -> Snake First Aid Guide.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Equip the general public with authoritative zoological parameters and comprehensive safety/preventative literature about different snake species.
  + **Interface:** ich media libraries, searchable encyclopedic UI, detailed taxonomy cards.
  + **Data processing:** Fetch structured JSON/CMS taxonomies, cache heavy assets locally.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.11.1 Snake Library Screen

![](data:image/png;base64...)

###### Figure 3.3.3.11.2 Snake Detail Screen

![](data:image/png;base64...)

###### Figure 3.3.3.11.3 Snake First Aid Guide Screen

* **Function details:**
  + **Data:** Species taxonomy databases, risk classification metrics, geographical habitats.
  + **Validation:** N/A (Mostly Read-only queries).
  + **Business rules:** Snake Detail views must prominently feature a direct CTA to that specific species' First Aid Guide to cut down navigation time in edge-case panics.
  + **Normal cases:** Member queries "Viper" -> Reads habitat detail -> Swipes to verify recommended first-aid.
  + **Abnormal cases:** Heavy network latency -> App serves last cached version of the Library to ensure availability.

#### 3.3.12 Blog List

* **Function trigger:** Member reads articles and community updates. Navigation path: Member Home -> Blog List -> Blog Detail.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Provide members with ongoing news, tips, and long-form articles related to snake safety and ecosystem awareness, while allowing community interaction via likes.
  + **Interface:** Long-form markdown blog readers, categorized lists of articles, and interactive interaction buttons (Like/Unlike).
  + **Data processing:** Fetch and parse markdown payloads safely, mutate and sync like states on the backend.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.12.1 Blog List Screen

![](data:image/png;base64...)

###### Figure 3.3.3.12.2 Blog Detail Screen (with unlike)

![](data:image/png;base64...)

###### Figure 3.3.3.12.3 Blog Detail Screen (with like)

* **Function details:**
  + **Data:** Raw markdown blog payloads, author details, publication dates, like counts, current user's like status.
  + **Validation:** User must be authenticated to toggle the like status.
  + **Business rules:**Blogs that mention specific snakes should visually link back to the Snake Library; Liking a blog updates the local UI optimistically while syncing to the server in the background.
  + **Normal cases:** Member opens Blog List -> Selects a recent article -> Reads the content -> Taps 'Like' button -> Like counter increments immediately.
  + **Abnormal cases:** Network failure prevents fetching new articles -> Shows offline cache; Backend fails to register Like -> UI reverts like toggle state and shows an error toast.

#### 3.3.13 Top-up

* **Function trigger:** Members need to add funds to their wallet. Navigation path: Profile Tab -> Top-up -> History Wallet.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Allow members to deposit funds into their SnakeAidPay wallet using external gateways.
  + **Interface:**Input fields for deposit amounts, payment gateway selection, and confirmation screens.
  + **Data processing:** Interface with 3rd-party Payment Processor APIs (e.g., PayOS), process webhooks to update ledger.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.13.1 Top-up Screen

* **Function details:**
  + **Data:** Fiat currency amount, payment intent tokens.
  + **Validation:** Top-up amounts must be greater than the minimum permitted value.
  + **Business rules:** Wallet balances are updated only after a successful webhook confirmation from the payment provider.
  + **Normal cases:** Member inputs 10,000 VND -> Pays via PayOS -> Webhook confirms -> Balance reflects change.
  + **Abnormal cases:** Payment gateway delays webhook -> UI marks transaction as "Processing" and polls until definitive state is reached.

#### 3.3.14 Withdraw

* **Function trigger:** Member wishes to extract funds from their wallet. Navigation path: Profile Tab -> Withdrawal -> History Wallet.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Allow members to submit a withdrawal request to transfer available balance from their SnakeAidPay wallet to a linked bank account, pending manual admin approval.
  + **Interface:** Bank account selection/input, withdrawal amount input, and confirmation dialogs indicating pending status.
  + **Data processing:** Mutate digital ledger states (lock funds), log withdrawal request for admin manual review and processing.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.14.1 Withdrawal Screen

![](data:image/png;base64...)

###### Figure 3.3.3.14.2 Withdrawal Request Modal

![](data:image/png;base64...)

###### Figure 3.3.3.14.3 Withdrawal Detail Screen (with status Pending)

![](data:image/png;base64...)

###### Figure 3.3.3.14.4 Withdrawal Detail Screen (with status Approved)

* **Function details:**
  + **Data:** Withdrawal amount, linked bank account details, request status (Pending/Approved/Rejected).
  + **Validation:** Withdrawal requests must exceed minimum systemic thresholds and cannot exceed available unheld balance.
  + **Business rules:** Funds are immediately locked from the available balance upon request, and the transaction is marked as "Pending”. The actual fiat transfer is handled manually by an Admin. Once transferred, the Admin updates the request to "Completed". If rejected, the locked funds are returned to the member's wallet.
  + **Normal cases:** Member requests withdrawal of 200,000 VND -> System locks funds and shows "Pending" -> Admin reviews and transfers money manually -> Status updates to "Completed".
  + **Abnormal cases:** Member tries to withdraw more than available -> UI blocks action; Admin rejects the withdrawal due to invalid bank details -> Funds are unlocked and returned to the member's balance.

#### 3.3.15 History Transaction

* **Function trigger:** Member wants to review past financial activity. Navigation path: Profile Tab -> History Transaction -> Transaction Detail.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Provide a transparent, auditable ledger of all deposits, withdrawals, and service payments.
  + **Interface:** Chronologically sorted list of transactions, filterable by type, with detailed receipt views.
  + **Data processing:** Fetch and paginate transactional records from the database.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.15.1 History Transaction Screen

![](data:image/png;base64...)

###### Figure 3.3.3.15.2 Transaction Detail Screen

* **Function details:**
  + **Data:** Unique transaction IDs, status enumerations (Pending, Completed, Failed), amounts, timestamps, transaction types.
  + **Validation:** N/A (Read-only view).
  + **Business rules:** All historical financial records are immutable.
  + **Normal cases:** Member opens History Transaction (“Lịch Sử Thanh Toán ”) button in Profile Tab -> Taps a recent payment -> Views detailed receipt.
  + **Abnormal cases:** Network drops during pagination -> UI shows retry button.

#### 3.3.16 Profile

* **Function trigger:** Member inspects or updates personal records. Navigation path: Profile Tab -> Edit Profile.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Authorize profile modifications and toggle application configurations.
  + **Interface:** Interactive form components.
  + **Data processing:** Perform CRUD operations on user schematics.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.16.1 Profile Tab Screen

![](data:image/png;base64...)

###### Figure 3.3.3.16.2 Edit Profile Screen

* **Function details:**
  + **Data:** Core demographic PII, application state preferences.
  + **Validation:** PII changes subject to strict regex constraints (Email/Phone format integrity).
  + **Business rules:** Profile updates sync across devices and sessions.
  + **Normal cases:** Member edit Profile -> Member save it -> UI re-renders instantly.
  + **Abnormal cases:** Validation fails on phone number update -> UI highlights invalid field.

#### 3.3.17 Activity History

* **Function trigger:** Member reviews past operations and engagements. Navigation path: Activity Tab -> Activity History -> Activity Detail.
* **Function description:**
  + **Actor:** Member.
  + **Purpose:** Maintain a rigorous audit trail of all historical engagements such as SOS requests, snake catching requests.
  + **Interface:** Complex chronologically sorted activity cards.
  + **Data processing:** Aggregate scattered microservice logs into unified Activity timelines.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.3.17.1 Activity Tab Screen

![](data:image/png;base64...)

###### Figure 3.3.3.17.2 Activity History Screen

![](data:image/png;base64...)

###### Figure 3.3.3.17.3 Activity Detail Screen

* **Function details:**
  + **Data:** Unified incident/consultation discrete historical payload objects.
  + **Validation:** N/A (Read-only view).
  + **Business rules:** Historical activity records are tightly bound and immutable (cannot be deleted by user for legal/audit safety reasons).
  + **Normal cases:** Member opens Activity Tab -> click button “Lịch sử” on header -> Taps one for details.
  + **Abnormal cases:** Network drops during pagination -> UI shows retry button.

###

###

### 3.4 Rescuer Portal Feature

#### 3.4.1 Rescuer Workspace & Notifications

* **Function trigger:** User successfully authenticated. Navigation path: Rescuer Login -> Rescuer Home <-> Notification / Work Schedule Tab
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Serve as the central mission control hub, providing daily operational stats and global alert management.
  + **Interface:** Dashboard with metrics, bottom navigation bar, active duty toggle, paginated notification list, and weekly schedule view.
  + **Data processing:** Fetch daily statistics, establish SignalR connection for live dispatch events, synchronize read/unread notification states, and load weekly work assignments. events, synchronize read/unread notification states.
* **Screen layout:**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.4.1.1 Rescuer Home And Notification Screen

![](data:image/png;base64...)

###### Figure 3.3.4.1.2 Work Schedule Tab Screen

* **Function details:**
  + **Data:** Rescuer ID, operational stats (completed missions), active duty status, notification payload array, schedule assignments.
  + **Validation:** Rescuers must hold an active JWT and valid role mapping.
  + **Business rules:** Notifications are marked read immediately upon interaction; Rescuers must toggle 'Active' to receive inbound dispatch events; schedule items are grouped by day and week.
  + **Normal cases:** Dashboard data loads seamlessly; Real-time dispatch alerts surface cleanly.
  + **Abnormal cases:** SignalR connection drops trigger silent background reconnects; Network partitions show offline indicators.

#### 3.4.2 Emergency Response Workflow

* **Function trigger:** System emits high-priority SOS dispatch. Navigation path: Rescuer Home (SOS Alert) -> Mission Detail - SOS -> Navigation Map -> On-scene Support -> Find Hospital -> Mission Completion -> Mission Success - Emergency.
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Orchestrate critical, time-sensitive emergency interventions including navigation, AI-backed first-aid, and medical facility routing.
  + **Interface:** High-contrast SOS modal, live multi-pin tracking map, AI clinical support cards, and hospital proximity directory.
  + **Data processing:** Bidirectional live GPS streaming via WebSocket, query AI triage models, fetch geospatial medical facility data.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.4.2.1 Mission Detail - SOS Screen

![](data:image/png;base64...)

###### Figure 3.3.4.2.2 Navigation Map Screen

![](data:image/png;base64...)

###### Figure 3.3.4.2.3 On-scene Support Screen

![](data:image/png;base64...)

###### Figure 3.3.4.2.4 Find Hospital Screen

![](data:image/png;base64...)

###### Figure 3.3.4.2.5 Mission Completion Screen

![](data:image/png;base64...)

###### Figure 3.3.4.2.6 Mission Success - Emergency Screen

* **Function details:**
  + **Data:** Patient live coordinates, clinical symptoms payload, AI first-aid recommendations, hospital geodata, mission resolution timestamp incident/consultation discrete historical payload objects.
  + **Validation:** Requires real-time GPS permissions and persistent telemetry connection.
  + **Business rules:** SOS missions explicitly override standard job queues; AI recommendations dynamically adjust based on mapped snake species/symptoms.
  + **Normal cases:** Rapid dispatch acceptance, precise patient location tracking, successful on-scene stabilization, and optional hospital handover.
  + **Abnormal cases:** Patient tracking telemetry cuts out (retains last known pin); AI recommendation endpoint degrades (shows cached generic first-aid).

####

#### 3.4.3 Snake Catching Workflow

* **Function trigger:** Rescuer selects an available job. Navigation path: Rescuer Home -> Available Jobs -> Request Detail -> Accept Request -> En Route -> Tracking -> Result Confirmation -> Mission Success - Snake Catching.
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Provide an end-to-end operational flow for accepting and executing non-emergency snake removal requests.
  + **Interface:** Tabular job list, detailed request context cards, live route mapping, camera capture for evidence, and fee breakdown summaries.
  + **Data processing:** Query geospatial job queues, calculate ETA via OSRM, process media evidence uploads, and submit finalized mission payloads.
* **Screen layout:**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.4.3.1 Request Catching Detail

![](data:image/png;base64...)

###### Figure 3.3.4.3.2 En Route Screen

![](data:image/png;base64...)

###### Figure 3.3.4.3.3 Tracking Screen

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.4.3.4 Result Confirmation & Mission Success - Snake Catching Screen

* **Function details:**
  + **Data:** Job request payload, target coordinates, equipment checklist state, photo evidence, confirmed snake species ID, calculated service fee.
  + **Validation:** Target location must be resolvable; Mandatory photo evidence required before mission closure.
  + **Business rules:** Acceptance binds the rescuer to the request SLA; System deducts platform commission from the final service fee.
  + **Normal cases:** Rescuer navigates to location, captures snake, uploads evidence, and system records successful mission closure.
  + **Abnormal cases:** OSRM routing fails gracefully to straight-line fallback; Evidence upload interruptions trigger retry queue.

#### 3.4.4 Profile & Feedback

* **Function trigger:** Navigation via global menubar. Navigation path: Rescuer Home -> Profile Tab -> Edit Profile / Feedback.
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Manage personal identity and view/update rating feedback.
  + **Interface:** Profile summaries, editable form fields, and rating feedback views.
  + **Data processing:** Retrieve and mutate profile records, aggregate rating data.
* **Screen layout:**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.4.4.1 Profile Tab Screen

![](data:image/png;base64...)

###### Figure 3.3.4.4.2 Feedback Screen

* **Function details:**
  + **Data:** Rescuer biographical data, review/rating payloads.
  + **Validation:** Input constraints on profile updates.
  + **Business rules:** Profile updates must preserve account identity and contact integrity; rating feedback is read-only history.
  + **Normal cases:** Rescuer updates profile avatar successfully.
  + **Abnormal cases:** Avatar media upload fails returning standard server error.

#### 3.4.5 Mission History

* **Function trigger:** Navigation via global menubar. Navigation path: Rescuer Home -> Mission History -> History Detail.
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Audit historical operational performance.
  + **Interface:** Historical timeline lists and mission detail views.
  + **Data processing:** Fetch paginated historical mission ledgers.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.4.5.1 Mission History Screen

![](data:image/png;base64...)

###### Figure 3.3.4.5.2 History Detail Screen

* **Function details:**
  + **Data:** Historical mission payloads (status).
  + **Validation:** None beyond authenticated access.
  + **Business rules:** Mission history ledgers are immutable read-only records.
  + **Normal cases:** Historical ledger loads deep pagination correctly.
  + **Abnormal cases:** Corrupted historical records render safe fallback states.

#### 3.4.6 Lessons

* **Function trigger:** Navigation via global menubar. Navigation path: Rescuer Home -> Lessons -> Lesson Detail.
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Provide educational training content for rescuer users
  + **Interface:** Categorized lesson lists and article viewers.
  + **Data processing:** Fetch static CMS content, process client-side search filtering, load media assets..
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.4.6.1 Lessons Screen

![](data:image/png;base64...)

###### Figure 3.3.4.6.2 Lesson Detail Screen

* **Function details:**
  + **Data:** Structured training content.
  + **Validation:** Search queries sanitize input strings.
  + **Business rules:** Educational content should remain accessible and concise for field use.
  + **Normal cases:** Rescuer opens a lesson from the home screen and reads the associated content successfully.
  + **Abnormal cases:** Remote media loading stalls in low-bandwidth areas (displays cached placeholders).

####

#### 3.4.7 Snake Library

* **Function trigger:** Navigation via global menubar. Navigation path: Rescuer Home -> Snake Library -> Snake Detail -> First Aid Guide.
* **Function description:**
  + **Actor:** Rescuer.
  + **Purpose:** Searchable species dictionary and structured first-aid protocol cards.
  + **Interface:** Categorized lesson lists and article viewers.
  + **Data processing:** Fetch taxonomy data, process species search filtering, load high-resolution imagery.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.4.7.1 Snake Library Screen (Rescuer)

![](data:image/png;base64...)

###### Figure 3.3.4.7.2 Snake Detail Screen (Rescuer)

![](data:image/png;base64...)

###### Figure 3.3.4.7.3 First Aid Guide (Rescuer)

* **Function details:**
  + **Data:** Species taxonomy databases, risk classification metrics, geographical habitats.
  + **Validation:** N/A (Mostly Read-only queries).
  + **Business rules:** Snake Detail views must prominently feature a direct CTA to that specific species' First Aid Guide to cut down navigation time in edge-case panics.
  + **Normal cases:** Rescuer queries "Viper" -> Reads habitat detail -> Swipes to verify recommended first-aid.
  + **Abnormal cases:** Heavy network latency -> App serves last cached version of the Library to ensure availability.

### 3.5 Expert Portal Feature

#### 3.5.1 Expert Home & Notification

* **Function trigger:** After successful expert login. From bottom navigation return actions.
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Central workspace to access profile, consultation, blog, knowledge, Notification and review tools.
  + **Interface:** Tab/navigation entry points and overview content.
  + **Data processing:** Load account context and module summaries for quick navigation.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.5.1.1 Home Screen

* **Function details:**
  + **Data:** Expert profile summary, consultation indicators, quick links.
  + **Validation:** Authenticated expert session is required.
  + **Business rules:** Expert Home is the hub entry to expert-only modules.
  + **Normal cases:** Home modules and navigation load successfully.
  + **Abnormal cases:** Partial module load failure with retry/fallback.

#### 3.5.2 Profile & Settings

* **Function trigger:** User opens profile/settings from Expert Home. Navigation path: Expert Home -> Profile Tab -> (Settings / Edit Profile / Working Hours / Withdraw).
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Maintain profile information, availability, service fees, account preferences, and wallet withdrawals.
  + **Interface:** Profile hub with settings/action menu and dedicated edit forms.
  + **Data processing:** Fetch current expert profile, update editable fields, save scheduling and fee configuration, submit withdrawal requests.
* **Screen layout:**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.5.2.1 Profile Tab Screen

![](data:image/png;base64...)

###### Figure 3.3.5.2.2 Withdraw Screen

![](data:image/png;base64...)

###### Figure 3.3.5.2.3 Working Hours Screen

* **Function details:**
  + **Data:** Profile: name, contact, biography, settings; Working hours: schedule slots and availability windows; Fees: scheduled consultation fee, emergency consultation fee; Wallet: withdrawal amount and payout information.
  + **Validation:** Editable fields must meet format constraints; Working-hour ranges must be valid; Fee values must be non-negative and within allowed limits; Withdrawal amount must not exceed available balance.
  + **Business rules:** Fee/schedule updates affect consultation booking behavior; Withdrawal flow must follow wallet and transaction constraints.
  + **Normal cases:** Profile/settings updates and withdrawal request submission succeed.
  + **Abnormal cases:** Validation failures; Insufficient wallet balance; API save or payout processing errors.

#### 3.5.3 Expert Feedback

* **Function trigger:** User opens feedback from Expert Profile. Navigation path: Profile Tab-> Feedback.
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Review user ratings and comments across completed consultations.
  + **Interface:** Feedback list with rating metadata and filtering.
  + **Data processing:** Fetch paginated review records and display summary indicators.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.5.3.1 Feedback Screen

* **Function details:**
  + **Data:** Rating score, review text, reviewer metadata, timestamp.
  + **Validation:** Valid filter/pagination parameters.
  + **Business rules:** Only feedback related to the authenticated expert is visible.
  + **Normal cases:** Feedback list loads and filters correctly.
  + **Abnormal cases:** No data, endpoint mismatch, network/API failure.

#### 3.5.4 AI Review Management

* **Function trigger:** User opens AI review queue from Expert Profile. Navigation path: Profile Tab -> AI Review Queue -> AI Review Detail.
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Verify AI recognition outcomes and make expert approval/rejection decisions.
  + **Interface:** Queue list and detail review panel.
  + **Data processing:** Load pending AI items, inspect detail payload, submit review decision.
* **Screen layout:**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.5.4.1 AI Review Screen

* **Function details:**
  + **Data:** Recognition result, confidence, media references, suggested species.
  + **Validation:** Item status must be reviewable; decision payload must be valid.
  + **Business rules:** Each review item should receive one final expert decision per review cycle.
  + **Normal cases:** Queue displays pending items and decisions are saved.
  + **Abnormal cases:** Stale item state, conflict on update, incomplete AI payload.

#### 3.5.5 Expert Knowledge Access

* **Function trigger:** User opens knowledge modules from Expert Home. Navigation path: Expert Home -> Snake Library / Expert Snake First Aid.
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Access snake species references and first-aid guidance for consultation support.
  + **Interface:** Species library pages and first-aid content screens.
  + **Data processing:** Load taxonomy/reference content and first-aid procedures.
* **Screen layout:**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.5.5.1 Snake Library and Snake Detail Screen

![](data:image/png;base64...)

###### Figure 3.3.5.5.2 Snake First Aid Screen

* **Data:** Species taxonomy databases, risk classification metrics, geographical habitats.
* **Validation:** N/A (Mostly Read-only queries).
* **Business rules:** Snake Detail views must prominently feature a direct CTA to that specific species' First Aid Guide to cut down navigation time in edge-case panics.
* **Normal cases:** Expert queries "Viper" -> Reads habitat detail -> Swipes to verify recommended first-aid.
* **Abnormal cases:** Heavy network latency -> App serves last cached version of the Library to ensure availability.

#### 3.5.6 Expert Blog Management

* **Function trigger:** User opens blog module from Expert Home. Navigation path: Expert Home -> Blog List -> Expert Blog Form.
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Create and manage expert-authored blog content.
  + **Interface:** Blog list with create/edit actions and blog form.
  + **Data processing:** Load blog list, submit create/update payload, update list state.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.5.6.1 Blog List Screen

![](data:image/png;base64...)

###### Figure 3.3.5.6.2 Expert Blog Form Screen

* **Function details:**
  + **Data:** Title, cover/media, category/tags, body content, publication status.
  + **Validation:** Required title/content and valid media payload.
  + **Business rules:** Post status follows editorial workflow (draft/pending/published as configured).
  + **Normal cases:** Blog entries are created/updated and reflected in list.
  + **Abnormal cases:** Validation errors, upload failures, moderation/state conflicts.

#### 3.5.7 Consultation Management

* **Function trigger:** User opens consultation list/history from Expert Home. Navigation path: Expert Home -> Consultation List/History -> Expert Consultation Detail -> Expert Video Waiting -> Video Consultation -> Expert Consultation Complete.
* **Function description:**
  + **Actor:** Expert.
  + **Purpose:** Manage consultation lifecycle from listing and detail inspection to live call, chat and completion.
  + **Interface:** History/list view, detail page, waiting room, live video session, in-room chat panel, completion summary.
  + **Data processing:** Load consultation records, open selected detail, initialize call session, initialize realtime chat channel, send/receive chat messages and attachments, persist completion outcome.
* **Screen layout:**

**![](data:image/png;base64...)![](data:image/png;base64...)**

###### Figure 3.3.5.7.1 Video Waiting and Video Consultation Screen

**![](data:image/png;base64...)![](data:image/png;base64...)**

###### Figure 3.3.5.7.2 Chat and Look Up Snake Species Screen

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 3.3.5.7.3 Consultation History Tab

* **Function details:**
  + **Data:** Consultation metadata, participant info, case context, call session state, chat message stream, message attachment URLs, completion summary.
  + **Validation:** Expert must have permission/assignment for selected consultation; Call join preconditions must be satisfied.
  + **Business rules:** Selecting an item in Consultation List/History opens Expert Consultation Detail; Consultation status transitions follow defined lifecycle.
  + **Normal cases:** The Expert can browse history, open details, join calls, and complete sessions.
  + **Abnormal cases:** Session expired/cancelled; Video connection/token failure; Completion update conflict.

#### 3.5.8 Global Emergency Consultation Handling

* **Function trigger:** System emits emergency consultation request while expert is online. Navigation path: Expert Global Emergency Popup Listener -> (Accept Emergency Request -> Expert Video Waiting) or (Reject Emergency Request).
* **Function description:**
  + **Actor:** Expert (with system event trigger).
  + **Purpose:** Allow rapid accept/reject actions for urgent consultation requests.
  + **Interface:** Global popup listener and accept/reject handling states.
  + **Data processing:** Receive real-time emergency event, submit acceptance/rejection, route accepted case to waiting room.
* **Screen layout:**

**![](data:image/png;base64...)**

###### Figure 3.3.5.8.1 Expert Global Emergency Popup Listener Screen

* **Function details:**
  + **Data:** Emergency request payload, requester context, dispatch status.
  + **Validation:** Request must still be active at action time.
  + **Business rules:** Accepted emergency requests route to Expert Video Waiting; Rejected requests must record final reject state.
  + **Normal cases:** Expert handles emergency request and state sync succeeds.
  + **Abnormal cases:** Request already assigned/expired; Real-time event lag or action submission failure.

###

### 3.6 Management Portal Feature

#### 3.6.1 Dashboard Management

* **Function trigger:**
  + After successful Admin login.
  + From sidebar/menu navigation back to dashboard.
* **Function description:**
  + **Actor:** Admin.
  + **Purpose:** Monitor top-level KPIs and operational trends.
  + **Interface:** KPI cards, revenue/cases/profit/commission charts, period filter, recent incidents/requests.
  + **Data processing:** Fetch analytics by period/range, aggregate dashboard data, support Excel export.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.1.1 Admin Dashboard

* **Function details:**
  + **Data:** Overview metrics, timeline charts, recent incidents/requests.
  + **Validation:** Period value must be valid (day/month/year).
  + **Business rules:** Dashboard is the entry point to all management modules.
  + **Normal cases:** KPIs and charts load successfully.
  + **Abnormal cases:** Partial widget load failure with fallback messaging and refresh.

#### 3.6.2 Workshifts Management

* **Function trigger:** Admin clicks “Quản lý lịch làm việc” from Sidebar.
* **Function description:** Manage shift templates, assignment calendar, rescuer details, single/bulk assignment, check-in/check-out flows.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.2.1 Workshifts Management Screen

* **Function details:**
  + **Data:** Shift templates, assignments, rescuer status.
  + **Validation:** Schedule conflict checks, valid rescuer role, valid shift time range.
  + **Business rules:** No overlapping assignments; check-in/check-out follows shift status.
  + **Normal cases:** Shift and assignment creation/update succeeds.
  + **Abnormal cases:** Scheduling conflicts, insufficient rescuer capacity, status update failures.

#### 3.6.3 Users Management

* **Function trigger:** Admin clicks “Quản lý người dùng” from Sidebar.
* **Function description:** Manage user list with role/status filters, detail panel view, and account-level actions.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.3.1 Users Management Screen

* **Function details:**
  + **Data:** User profiles, status, role, growth analytics.
  + **Validation:** Valid filter parameters and permission-gated actions.
  + **Business rules:** Account status changes should be auditable.
  + **Normal cases:** View/filter/update user status successfully.
  + **Abnormal cases:** Permission denied or data synchronization errors.

#### 3.6.4 Request Management

* **Function trigger:** User clicks “quản lý sự cố/quản lý yêu cầu bắt rắn/quản lý phiên tư vấn” from Sidebar.
* **Function description:** Manage all incoming requests in one unified module, including Incident, Snake Catching, and Consultation requests, with keyword search, type/status filters, pagination, and action-column detail flow. Detail view uses the corresponding request-type detail endpoint to load full payload before rendering.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.4.1 Incidents Management Screen

![](data:image/png;base64...)

###### Figure 3.3.6.4.2 Snake Catching Management Screen

![](data:image/png;base64...)

###### Figure 3.3.6.4.3 Consultations Management Screen

* **Function details:**
  + **Data:** Unified request summary, requester profile, contact/location data, assigned operator or rescuer data, request-type-specific payload (incident info, snake details, consultation content), related media, and audit fields.
  + **Validation:** Valid request type and request ID for detail endpoint; valid query/filter values.
  + **Business rules:** Detail modal must always prefer detail endpoint payload over list row snapshot. Request-type-specific fields are rendered conditionally based on request type.
  + **Normal cases:** List loads successfully, users filter by request type/status, detail opens via action button, and all sections render correctly for each request type.
  + **Abnormal cases:** Detail endpoint failure, unsupported request type, missing nested fields, empty detail/media arrays, or partially returned payload.

#### 3.6.5 Snakes Management

* **Function trigger:** Admin clicks “Quản lý loài rắn” from Sidebar.
* **Function description:** Manage snake species records, upsert data, antivenom links, and prevalence map references.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.5.1 Snakes Management Screen

* **Function details:**
  + **Data:** Species profile, taxonomy, prevalence, linked antivenoms.
  + **Validation:** Required species fields and naming constraints.
  + **Business rules:** Antivenom mapping must reference valid catalog items.
  + **Normal cases:** Species create/update succeeds.
  + **Abnormal cases:** Duplicate species or invalid mapping references.

#### 3.6.6 Antivenoms Management

* **Function trigger:** Admin clicks “Quản lý huyết thanh” from Sidebar.
* **Function description:** Manage antivenom catalog with list/detail/upsert flows.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.6.1 Antivenoms Management

* **Function details:**
  + **Data:** Product name, manufacturer, dosage/form, mapping metadata.
  + **Validation:** Required fields and uniqueness constraints.
  + **Business rules:** Antivenom data is reused across related modules.
  + **Normal/Abnormal cases:** CRUD success or duplicate/validation errors.

#### 3.6.7 Treatment Facilities Management

* **Function trigger:** Admin clicks “Quản lý cơ sở điều trị” from Sidebar.
* **Function description:** Manage treatment facilities with detail panel, upsert form, and antivenom associations.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.7.1 Treatment Facilities Management Screen

* **Function details:**
  + **Data:** Facility profile, address, contact details, available antivenoms.
  + **Validation:** Valid contact/address formats and required fields.
  + **Business rules:** Antivenom associations must reference valid items.
  + **Normal/Abnormal cases:** Successful save or validation/reference errors.

#### 3.6.8 Transactions & Withdrawals Management

* **Function trigger:** Admin clicks “Quản lý giao dịch” from Sidebar.
* **Function description:** Financial operations page with two internal tabs: Transactions and Withdrawals. Supports transaction filtering, paging, detail modal, and withdrawal processing flow.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.8.1 Transactions Screen

![](data:image/png;base64...)

###### Figure 3.3.6.8.2 Withdrawals Screen

* **Function details:**
  + **Data:** Transaction records, withdrawal requests, status/history.
  + **Validation:** Valid filter parameters (type, keyword, pagination).
  + **Business rules:** Withdrawal processing follows state transitions and audit requirements.
  + **Normal cases:** Lookup/detail/review flows complete successfully.
  + **Abnormal cases:** Withdrawal condition failures or status update errors.

#### 3.6.9 Settings Management

* **Function trigger:** Admin clicks “Quản lý cấu hình động” from Sidebar.
* **Function description:** Manage dynamic system settings (list/search/detail/create-edit modal).
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.9.1 Settings Management Screen

* **Function details:**
  + **Data:** key, value, valueType (String/Int/Decimal/Boolean/Json), metadata.
  + **Validation:** Value format must match selected data type.
  + **Business rules:** Setting key is unique; updates can affect runtime behavior.
  + **Normal/Abnormal cases:** Successful update or parse/validation errors.

#### 3.6.10 Report Media Management

* **Function trigger:** Admin clicks “Quản lý ảnh báo cáo rắn” from Sidebar.
* **Function description:** Manage multi-source report media (Community Report, Snakebite Incident, Rescue Mission, Snake Catching Request/Mission). Supports status/reference/date filters, media list, and recognition detail modal.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.10.1 Report Media Management Screen

* **Function details:**
  + **Data:** Report media, recognition status, confidence, reference type, expert review state.
  + **Validation:** Valid date range and confidence range.
  + **Business rules:** Media progresses through recognition states (processing/completed/failed/expert verified/expert rejected).
  + **Normal cases:** Filtering and detail inspection succeed.
  + **Abnormal cases:** No results, detail load failure, incomplete recognition payload.

#### 3.6.11 Lessons Management

* **Function trigger:** Admin clicks “Quản lý bài học” from Sidebar.
* **Function description:** Lesson CMS with category/publish filters, list-detail view, upsert modal, and preview.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.11.1 Lessons Management Screen

* **Function details:**
  + **Data:** Lesson metadata, content, publish state.
  + **Validation:** Required title/content/category fields.
  + **Business rules:** Draft/published workflow applies.
  + **Normal/Abnormal cases:** Create/update/publish success or validation/save errors.

#### 3.6.12 Blogs Management

* **Function trigger:** Admin clicks “Quản lý bài viết” from Sidebar.
* **Function description:** Blog CMS with status tabs, list-detail view, upsert modal, preview, and approve/reject moderation.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.6.12.1 Blogs Management Screen

* **Function details:**
  + **Data:** Blog metadata/content, status, rejection reason.
  + **Validation:** Valid title/content/category/tags; rejection requires reason.
  + **Business rules:** Workflow is Draft -> PendingApproval -> Published/Rejected.
  + **Normal cases:** Create/update/review/status transitions succeed.
  + **Abnormal cases:** Rejection without reason, status update failure, update conflicts.

### 3.7 Operator Portal Feature

#### 3.7.1 Operator Dashboard

* **Function trigger:**
  + After successful Operator login.
  + This is the only currently implemented Operator route.
* **Function description:**
  + **Actor:** Operator.
  + **Purpose:** Receive, verify, and dispatch incidents/requests in near real time.
  + **Interface:** Live map, incident/request panels, shift schedule panel, pending alerts, incident/request detail modals.
  + **Data processing:** Real-time event sync (hub/events), assignment/dispatch actions, state refresh on incoming events.
* **Screen layout:**

![](data:image/png;base64...)

###### Figure 3.3.7.1.1 Operator Dashboard

![](data:image/png;base64...)

###### Figure 3.3.7.1.2 Request Detail Modal

* **Function details:**
  + **Data:** Live incidents, catching requests, online rescuers, shift assignments.
  + **Validation:** Assignment/dispatch requires valid rescuer and compatible current status.
  + **Business rules:** Incident/request lifecycle is enforced; pending/abort alerts are prioritized.
  + **Normal cases:** Incident intake and dispatch flow succeeds.
  + **Abnormal cases:** Rescuer abort, false alarm, delayed/failed real-time events, state conflicts.

##

## 4. Non-Functional Requirements

### 4.1 External Interfaces

The system shall interact with the following external systems and interfaces:

* **Mobile/Web Clients:**
  + Support responsive web interface and mobile applications.
  + Compatible with modern browsers (Chrome, Edge, Safari).
* **Map & Geolocation Services:**
  + Integrate with third-party map services for location tracking and routing.
  + Support real-time GPS tracking for rescue missions.
* **AI Recognition Service:**
  + Integrate with deployed AI models for snake identification from uploaded images.
  + Accept image input and return predicted species with confidence score.
* **Payment Gateway:**
  + Support integration with third-party payment providers for consultation and service payments.
* **Notification Services:**
  + Integrate with push notification services (e.g., Firebase Cloud Messaging) for real-time alerts.

### 4.2 Quality Attributes

#### 4.2.1 Usability

* First-time users can use the system with minimal guidance (≤ 10 minutes).
* Users can:
  + Report a snakebite incident within ≤ 60 seconds.
  + Create a snake-catching request within ≤ 90 seconds.
* UI is simple, clear, and supports Vietnamese (primary language).
* Error messages must be clear and understandable.

#### 4.2.2 Performance

* API response time:
  + Average ≤ 500 ms
  + Maximum ≤ 10 seconds
* AI image processing time ≤ 7 seconds.
* System supports at least 200–300 concurrent users.

#### 4.2.3 Reliability & Availability

* System uptime ≥ 99%.
* Core features (snakebite reporting) must always be available.
* Basic error handling must prevent system crashes.

#### 4.2.4 Security

* All data is transmitted via HTTPS.
* Authentication uses a token-based mechanism (e.g., JWT).
* Passwords must be hashed.
* Prevent common vulnerabilities:
  + SQL Injection
  + XSS
* Role-based access control (Member / Rescuer / Expert).

#### 4.2.5 Scalability

* System architecture allows future scaling (e.g., adding more users or services).
* Database supports indexing for performance optimization.

#### 4.2.6 Maintainability

* Code follows clean architecture and modular design.
* Logging is implemented for key actions (login, request creation, assignment).
* System can be updated without affecting core features.

## 5. Requirement Appendix

### 5.1 Business Rules

| **ID** | **Rule Definition** |
| --- | --- |
| BR-01 | Each account must be associated with exactly one primary role (Member, Rescuer, Expert, or Operator). |
| BR-02 | A user email must be unique across all active accounts. |
| BR-03 | A phone number must be unique across all active accounts. |
| BR-04 | An account must verify email before accessing protected features. |
| BR-05 | An account in Suspended status cannot create any new requests. |
| BR-06 | A member can only manage resources that they own. |
| BR-07 | A rescuer must have an approved profile before receiving assignments. |
| BR-08 | An expert must have a verified qualification before consultation is enabled. |
| BR-09 | An operator can assign missions only within their authorized region. |
| BR-10 | An admin can lock or unlock any non-admin account. |
| BR-11 | A snake-catching request must include a valid location and contact phone. |
| BR-12 | A request location must contain latitude and longitude within valid ranges. |
| BR-13 | A new emergency request is created with Pending status by default. |
| BR-14 | A request can be canceled by its owner only while status is Pending or Accepted. |
| BR-15 | A canceled request cannot transition to InProgress status. |
| BR-16 | A completed request cannot be modified except by admin audit actions. |
| BR-17 | A mission must be assigned to exactly one primary rescuer at a time. |
| BR-18 | A rescuer cannot accept two overlapping missions in time. |
| BR-19 | Mission start time must be earlier than mission completion time. |
| BR-20 | Only the assigned rescuer can mark mission arrival at the scene. |
| BR-21 | Proof-of-arrival requires at least one timestamped evidence record. |
| BR-22 | Mission completion requires a completion note and outcome classification. |
| BR-23 | A mission outcome must be one of Resolved, Escalated, or UnableToResolve. |
| BR-24 | Escalated outcomes require an escalation reason and destination team. |
| BR-25 | A request cannot be reassigned if mission status is Completed. |
| BR-26 | Operator reassignment must preserve full assignment history. |
| BR-27 | All status changes must write an immutable audit log entry. |
| BR-28 | Audit logs must include actor id, timestamp, and action type. |
| BR-29 | Soft-deleted records must not appear in default list endpoints. |
| BR-30 | Hard delete is restricted to data-retention cleanup jobs only. |
| BR-31 | A consultation booking must select an available expert slot. |
| BR-32 | Expert availability slots cannot overlap for the same expert. |
| BR-33 | A consultation can be booked only if payment is authorized. |
| BR-34 | A consultation can be canceled free of charge before the grace cutoff. |
| BR-35 | Late cancellation may apply a configurable cancellation fee. |
| BR-36 | Consultation session token is generated only after booking confirmation. |
| BR-37 | Session tokens must expire after the configured validity duration. |
| BR-38 | Only participants of a consultation can join its call room. |
| BR-39 | A chat message must belong to exactly one conversation thread. |
| BR-40 | Deleted chat messages must remain visible to moderators in audit mode. |
| BR-41 | Message attachments must pass virus scan before becoming downloadable. |
| BR-42 | Unsupported file types must be rejected at upload time. |
| BR-43 | Uploaded images must not exceed the configured maximum size limit. |
| BR-44 | AI snake recognition can run only on approved image mime types. |
| BR-45 | AI recognition results must include confidence score and model version. |
| BR-46 | Low-confidence AI results must be flagged for expert review. |
| BR-47 | Only experts can publish final species verification decisions. |
| BR-48 | Species verification updates must keep previous decision history. |
| BR-49 | A snake species record must have exactly one scientific name. |
| BR-50 | Duplicate species records with the same scientific name are not allowed. |
| BR-51 | A rescue shift must have start and end times within one calendar day. |
| BR-52 | A rescuer cannot be assigned to two shifts that overlap. |
| BR-53 | Shift assignment changes must notify affected rescuers immediately. |
| BR-54 | A rescuer marked OffDuty cannot receive new dispatches. |
| BR-55 | Dispatch priority must be computed using urgency and proximity rules. |
| BR-56 | Critical urgency requests must bypass normal queue ordering. |
| BR-57 | Operators can manually override dispatch orders with justification. |
| BR-58 | Manual dispatch override must be logged with reason code. |
| BR-59 | A wallet can only be created for active and verified accounts. |
| BR-60 | Each account can own at most one wallet in active state. |
| BR-61 | Wallet balance cannot become negative after any transaction. |
| BR-62 | All wallet debits and credits must be represented by ledger entries. |
| BR-63 | Ledger entries are immutable once committed. |
| BR-64 | Refund transactions must reference the original payment transaction. |
| BR-65 | A payment transaction must have exactly one terminal status. |
| BR-66 | Terminal payment statuses are Succeeded, Failed, Refunded, and Canceled. |
| BR-67 | A failed payment cannot be converted directly to Succeeded. |
| BR-68 | Idempotency key is required for payment creation operations. |
| BR-69 | Repeated payment requests with the same idempotency key must return the same result. |
| BR-70 | Withdrawal requests require a verified bank account binding. |
| BR-71 | A bank account can be bound to one user at a time unless unbound. |
| BR-72 | Withdrawal amount must be greater than zero and within daily limits. |
| BR-73 | Daily withdrawal limit is evaluated by account and local timezone. |
| BR-74 | Insufficient available balance must reject withdrawal creation. |
| BR-75 | Pending withdrawal amount must be reserved from available balance. |
| BR-76 | Withdrawal approval can be performed only by authorized admin roles. |
| BR-77 | A rejected withdrawal must release the reserved amount immediately. |
| BR-78 | A completed withdrawal cannot be canceled by the requester. |
| BR-79 | All withdrawal state transitions must be tracked in timeline history. |
| BR-80 | KYC verification is required before the first withdrawal request. |
| BR-81 | Insurance claim requests must reference a completed mission. |
| BR-82 | Claim amount cannot exceed policy coverage cap for the period. |
| BR-83 | Claim review requires at least one evidence document. |
| BR-84 | Approved claims must create a payout transaction record. |
| BR-85 | A policy cannot be active before premium payment succeeds. |
| BR-86 | Policy renewal must preserve continuity unless explicitly terminated. |
| BR-87 | A notification must target at least one valid recipient channel. |
| BR-88 | Push notifications for critical alerts must include fallback SMS policy. |
| BR-89 | Notification retries must respect exponential backoff configuration. |
| BR-90 | Read receipts can only be marked by the notification recipient. |
| BR-91 | Geolocation updates must include timestamp and accuracy metadata. |
| BR-92 | Location pings older than the retention threshold are archived. |
| BR-93 | Map route estimation must use latest confirmed destination coordinates. |
| BR-94 | Emergency hotspot analytics must aggregate by configured spatial grid. |
| BR-95 | Reports must exclude personally identifiable information by default. |
| BR-96 | Exporting personally identifiable data requires elevated permission. |
| BR-97 | CSV exports must include generation timestamp and filter criteria. |
| BR-98 | System timestamps must be stored in UTC internally. |
| BR-99 | Client-facing datetime values must include timezone offset in API responses. |
| BR-100 | Password reset tokens must be single-use and time-limited. |
| BR-101 | Login attempts exceeding threshold must trigger temporary account lockout. |
| BR-102 | Unlock after lockout requires either cooldown expiry or admin action. |
| BR-103 | Refresh tokens must be revocable individually per device session. |
| BR-104 | Revoked tokens must be denied even if not yet expired. |
| BR-105 | Any role change must force re-authentication for all active sessions. |
| BR-106 | Public endpoints must not return internal identifiers unless required. |
| BR-107 | API error responses must use standardized code and message schema. |
| BR-108 | Validation errors must include field-level details for client correction. |
| BR-109 | Rate limits must be enforced per client and endpoint policy. |
| BR-110 | The deposit for catching service is non-refundable under any circumstances. |
| BR-111 | If the rescuer reports that no snake is found at the scene, the customer will not be charged the final price. |

###### Table 3.5.1.1 Business Rules

### 5.2 Common Requirements

The system must ensure stable operation and support the core activities of rescue management. All users should be able to access system functions according to their assigned roles and permissions. The system should maintain data consistency and accuracy during operations, while also ensuring that important activities are recorded for monitoring and management purposes.

In addition, the system must allow administrators to configure necessary settings when required. Data security and privacy must also be considered, especially when handling sensitive information. The system should support timely updates and smooth performance to ensure that emergency operations can be managed effectively.

### 5.3 Application Messages List

| **#** | **Message code** | **Message Type** | **Context** | **Content** |
| --- | --- | --- | --- | --- |
| 1 | MSG01 | Inline | No snake species found after search | No snake species found. |
| 2 | MSG02 | Inline (Error) | Required field is empty | The \* field is required. |
| 3 | MSG03 | Toast | Snakebite incident created successfully | Snakebite incident reported successfully. |
| 4 | MSG04 | Toast | Snake-catching request created successfully | Snake-catching request submitted successfully. |
| 5 | MSG05 | Toast | Rescuer assigned successfully | The rescuer has been assigned successfully. |
| 6 | MSG06 | Toast | Mission completed | Rescue mission completed successfully. |
| 7 | MSG07 | Toast | Image uploaded for AI recognition | Image uploaded successfully. |
| 8 | MSG08 | Inline (Error) | Input exceeds max length | Exceed max length of {max\_length}. |
| 9 | MSG09 | Inline (Error) | Login failed | Incorrect username or password. |
| 10 | MSG10 | Toast | Consultation booked | Consultation booked successfully. |
| 11 | MSG11 | Inline | No consultation slot available | No available slot for selected date. |
| 12 | MSG12 | Toast | Profile updated | Profile updated successfully. |
| 13 | MSG13 | Inline (Error) | Invalid phone number | Phone number format is invalid. |
| 14 | MSG14 | Inline (Error) | Invalid email | Email format is invalid. |
| 15 | MSG15 | Toast | Password changed | Password changed successfully. |
| 16 | MSG16 | Inline (Error) | Current password mismatch | The current password is incorrect. |
| 17 | MSG17 | Toast | Forgot password email sent | Password reset email has been sent. |
| 18 | MSG18 | Inline (Error) | Reset token expired | The reset link has expired. |
| 19 | MSG19 | Toast | Email verified | Email verified successfully. |
| 20 | MSG20 | Inline (Error) | OTP invalid | Invalid OTP code. |
| 21 | MSG21 | Inline (Error) | OTP expired | OTP has expired. |
| 22 | MSG22 | Toast | OTP resent | OTP presented successfully. |
| 23 | MSG23 | Toast | Logout successful | You have logged out successfully. |
| 24 | MSG24 | Inline (Error) | Unauthorized access | You are not authorized to perform this action. |
| 25 | MSG25 | Inline (Error) | Forbidden resource | You do not have permission to access this resource. |
| 26 | MSG26 | Banner | System maintenance | The system is under maintenance. Please try again later. |
| 27 | MSG27 | Toast | Report submitted | Your report has been submitted. |
| 28 | MSG28 | Inline (Error) | Duplicate report | This incident has already been reported. |
| 29 | MSG29 | Toast | Location detected | Current location captured successfully. |
| 30 | MSG30 | Inline (Error) | Location permission denied | Please enable location permission. |
| 31 | MSG31 | Inline (Error) | Location unavailable | Unable to detect current location. |
| 32 | MSG32 | Toast | Map loaded | Map loaded successfully. |
| 33 | MSG33 | Inline (Error) | Address not found | Unable to find this address. |
| 34 | MSG34 | Toast | Request accepted | Request accepted successfully. |
| 35 | MSG35 | Toast | Request canceled | Request canceled successfully. |
| 36 | MSG36 | Inline (Error) | Cannot cancel request | Requests cannot be canceled at this stage. |
| 37 | MSG37 | Toast | Mission started | The mission started successfully. |
| 38 | MSG38 | Toast | Arrival confirmed | Arrival at the scene confirmed. |
| 39 | MSG39 | Inline (Error) | Evidence required | At least one evidence image is required. |
| 40 | MSG40 | Toast | Evidence uploaded | Evidence uploaded successfully. |
| 41 | MSG41 | Inline (Error) | Unsupported file type | File type is not supported. |
| 42 | MSG42 | Inline (Error) | File size exceeded | File exceeds maximum allowed size. |
| 43 | MSG43 | Toast | AI recognition completed | AI recognition completed successfully. |
| 44 | MSG44 | Inline | Low confidence result | Low confidence result. Expert review is recommended. |
| 45 | MSG45 | Toast | Species verified | Species verification completed. |
| 46 | MSG46 | Inline (Error) | Verification failed | Unable to verify species at this time. |
| 47 | MSG47 | Toast | Shift created | Shift created successfully. |
| 48 | MSG48 | Toast | Shift updated | Shift updated successfully. |
| 49 | MSG49 | Inline (Error) | Shift overlap | Shift overlaps with an existing assignment. |
| 50 | MSG50 | Toast | Shift assignment sent | Shift assigned to rescuer successfully. |
| 51 | MSG51 | Inline (Error) | Rescuer off duty | The selected rescuer is currently off duty. |
| 52 | MSG52 | Toast | Dispatch created | Dispatch created successfully. |
| 53 | MSG53 | Inline (Error) | No available rescuer | No available rescuers in the selected area. |
| 54 | MSG54 | Toast | Dispatch override applied | Dispatch order override saved. |
| 55 | MSG55 | Inline (Error) | Override reason required | Override reason is required. |
| 56 | MSG56 | Toast | Wallet created | Wallet created successfully. |
| 57 | MSG57 | Toast | Top-up successful | Wallet top-up successful. |
| 58 | MSG58 | Inline (Error) | Top-up failed | Unable to process top-up. |
| 59 | MSG59 | Inline (Error) | Insufficient balance | Insufficient wallet balance. |
| 60 | MSG60 | Toast | Payment successful | Payment completed successfully. |
| 61 | MSG61 | Inline (Error) | Payment failed | Payment could not be completed. |
| 62 | MSG62 | Inline (Error) | Payment canceled | Payment has been canceled. |
| 63 | MSG63 | Inline (Error) | Duplicate payment request | Duplicate payment request detected. |
| 64 | MSG64 | Toast | Refund successful | Refund processed successfully. |
| 65 | MSG65 | Inline (Error) | Refund failed | Refunds could not be processed. |
| 66 | MSG66 | Toast | Bank account linked | Bank account linked successfully. |
| 67 | MSG67 | Inline (Error) | Bank account verification failed | Bank account verification failed. |
| 68 | MSG68 | Toast | Withdrawal request submitted | Withdrawal request submitted successfully. |
| 69 | MSG69 | Inline (Error) | Withdrawal limit exceeded | Daily withdrawal limit exceeded. |
| 70 | MSG70 | Inline (Error) | Withdrawal rejected | The withdrawal request was rejected. |
| 71 | MSG71 | Toast | Withdrawal approved | Withdrawal approved successfully. |
| 72 | MSG72 | Toast | Withdrawal completed | Withdrawal completed successfully. |
| 73 | MSG73 | Inline (Error) | KYC required | KYC verification is required before withdrawal. |
| 74 | MSG74 | Toast | KYC submitted | KYC documents submitted successfully. |
| 75 | MSG75 | Inline (Error) | KYC rejected | KYC verification was rejected. |
| 76 | MSG76 | Toast | Policy activated | Insurance policy activated successfully. |
| 77 | MSG77 | Inline (Error) | Policy activation failed | Unable to activate policy. |
| 78 | MSG78 | Toast | Claim submitted | Insurance claim submitted successfully. |
| 79 | MSG79 | Inline (Error) | Claim evidence missing | Please upload required claim evidence. |
| 80 | MSG80 | Toast | Claim approved | Insurance claim approved. |
| 81 | MSG81 | Inline (Error) | Claim rejected | The insurance claim was rejected. |
| 82 | MSG82 | Toast | Notification sent | Notification sent successfully. |
| 83 | MSG83 | Inline (Error) | Notification failed | Unable to send notification. |
| 84 | MSG84 | Toast | Push token updated | Push token updated successfully. |
| 85 | MSG85 | Inline (Error) | Push token invalid | Push token is invalid. |
| 86 | MSG86 | Toast | Message sent | Message sent successfully. |
| 87 | MSG87 | Inline (Error) | Message send failed | Unable to send messages. |
| 88 | MSG88 | Inline (Error) | Empty message | Message content cannot be empty. |
| 89 | MSG89 | Toast | Conversation archived | Conversation archived successfully. |
| 90 | MSG90 | Inline (Error) | Conversation not found | Conversation was not found. |
| 91 | MSG91 | Toast | Call started | The video call started successfully. |
| 92 | MSG92 | Inline (Error) | Call connection failed | Unable to connect to call. |
| 93 | MSG93 | Toast | Call ended | The video call ended. |
| 94 | MSG94 | Inline (Error) | Microphone permission denied | Please allow microphone access. |
| 95 | MSG95 | Inline (Error) | Camera permission denied | Please allow camera access. |
| 96 | MSG96 | Toast | Admin action saved | Admin action completed successfully. |
| 97 | MSG97 | Inline (Error) | Invalid admin input | Please check admin form input. |
| 98 | MSG98 | Toast | User suspended | User account suspended successfully. |
| 99 | MSG99 | Toast | User unsuspended | User account unsuspended successfully. |
| 100 | MSG100 | Inline (Error) | Cannot suspend admin | Admin account cannot be suspended. |
| 101 | MSG101 | Toast | Role updated | User role updated successfully. |
| 102 | MSG102 | Inline (Error) | Role update failed | Unable to update user role. |
| 103 | MSG103 | Toast | Data exported | Data exported successfully. |
| 104 | MSG104 | Inline (Error) | Export failed | Failed to export data. |
| 105 | MSG105 | Toast | Import completed | Data imported successfully. |
| 106 | MSG106 | Inline (Error) | Import file invalid | Import file format is invalid. |
| 107 | MSG107 | Inline (Error) | Rate limit exceeded | Too many requests. Please try again later. |
| 108 | MSG108 | Banner | Server unavailable | Service is temporarily unavailable. |
| 109 | MSG109 | Inline (Error) | Request timeout | Request timed out. Please retry. |

###### Table 3.5.3.1 Application Messages List

# IV. Software Design Description

## 1. System Design

### 1.1 System Architecture

###

![](data:image/png;base64...)

###### Figure 4.1.1.1 System Architecture Diagram

### 1.2 Package Diagram

![](data:image/png;base64...)

###### Figure 4.1.1.2 Package Diagram

***Package Descriptions***

| **No** | **Package** | **Description** |
| --- | --- | --- |
| 01 | SnakeAid.Api | The entry point for HTTP/API requests. Contains controllers, dependency injection setup, SignalR hubs, and static files. Responsible for handling client requests and returning responses. |
| 02 | SnakeAid.Service | Implements business logic and application workflows. Contains service interfaces and their implementations, as well as internal utilities and options. Acts as the main bridge between API and data access layers. |
| 03 | SnakeAid.Repository | Handles data access and persistence. Contains repository interfaces and implementations, database migrations, data providers, and seed data. |
| 04 | SnakeAid.Core | Contains core domain models, enums, request/response DTOs, constants, utilities, validators, exceptions, and mappings. |
| 05 | SnakeAid.Api.Controllers | API endpoints that handle HTTP requests and responses. |
| 06 | SnakeAid.Api.Hub | SignalR hubs for real-time communication. |
| 07 | SnakeAid.Api.DI | Dependency injection setup and service registrations. |
| 08 | SnakeAid.Api.Middleware | Middleware setup and registrations. |
| 09 | SnakeAid.Service.Interfaces | Service interface definitions for dependency injection and abstraction. |
| 10 | SnakeAid.Service.Implements | Concrete implementations of service interfaces containing business logic. |
| 11 | SnakeAid.Service.Consumers | Background consumers or event handlers. |
| 12 | SnakeAid.Service.Extensions | Extension methods for service layer utilities. |
| 13 | SnakeAid.Repository.Data | Database context and class configurations. |
| 14 | SnakeAid.Repository.Interfaces | Repository interface definitions for data access abstraction. |
| 15 | SnakeAid.Repository.Implements | Concrete repository implementations for data persistence. |
| 16 | SnakeAid.Repository.Migrations | Database migration scripts and history. |
| 17 | SnakeAid.Core.Entities | Core domain models/entities representing business data. |
| 18 | SnakeAid.Core.Request | Data transfer objects (DTOs) for incoming API requests. |
| 19 | SnakeAid.Core.Response | DTOs for outgoing API responses. |
| 20 | SnakeAid.Core.Constants | Application-wide constant values. |
| 21 | SnakeAid.Core.Utils | Utility classes and helper functions. |
| 22 | SnakeAid.Core.Validators | Validation logic for models and requests. |

###### Table 4.1.1.3 Package Description

## 2. Database Design

###### Figure 4.2.1 Physical Database![](data:image/png;base64...)

**Table Descriptions**

| **No** | **Table** | **Description** |
| --- | --- | --- |
| 1 | Accounts | Stores the main user account, authentication, and role information used across the platform.  - Primary key: Id.  - Foreign keys: none. |
| 2 | AIModels | Stores metadata about deployed AI model versions used for snake recognition.  - Primary key: Id.  - Foreign keys: none. |
| 3 | AISnakeClassMappings | Maps AI model output classes to snake species so raw AI detections can be interpreted by the application.  - Primary key: Id.  - Foreign keys: SnakeSpeciesId, AIModelId. |
| 4 | Antivenoms | Stores antivenom master data and links each antivenom to its treatment facility.  - Primary key: Id.  - Foreign keys: TreatmentFacilityId. |
| 5 | AppNotifications | Stores in-app notifications sent to users.  - Primary key: Id.  - Foreign keys: UserId. |
| 6 | Blogs | Stores blog posts or educational articles authored by experts.  - Primary key: Id.  - Foreign keys: AuthorId. |
| 7 | CatchingEnvironment | Stores reference categories describing the environment in which a snake catching mission takes place.  - Primary key: Id.  - Foreign keys: none. |
| 8 | CatchingMissionDetails | Stores species and quantity detail lines recorded under a snake catching mission.  - Primary key: Id.  - Foreign keys: SnakeCatchingMissionId, SnakeSpeciesId. |
| 9 | CatchingRequestDetails | Stores species and quantity detail lines recorded under a snake catching request.  - Primary key: Id.  - Foreign keys: SnakeCatchingRequestId, SnakeSpeciesId. |
| 10 | ChatMessages | Stores chat messages exchanged during a consultation session.  - Primary key: Id.  - Foreign keys: ConsultationId, SenderId. |
| 11 | CommunityReports | Stores community-submitted snake sighting or incident reports with location, notes, optional identified species, and attached media.  - Primary key: Id.  - Foreign keys: UserId. |
| 12 | ConsultationBookings | Stores scheduled consultation bookings, including user, expert, slot, consultation link, pricing, and payment-related status.  - Primary key: Id.  - Foreign keys: UserId, ExpertId, ConsultationId, TimeSlotId. |
| 13 | Consultations | Stores core consultation session records for scheduled and emergency consultations, including participants, room, timing, type, and status.  - Primary key: Id.  - Foreign keys: CallerId, CalleeId, ExpertTimeSlotId. |
| 14 | ExpertCertificates | Stores certificate and verification records submitted by experts to prove qualifications.  - Primary key: Id.  - Foreign keys: ExpertId in current code, though it is not marked as an FK in the ERD. |
| 15 | ExpertProfiles | Stores expert-specific profile information such as biography, online status, fee, and rating metrics.  - Primary key: AccountId.  - Foreign keys: AccountId. |
| 16 | ExpertSpecializations | Stores the many-to-many relationship between experts and their specializations.  - Primary key: Id.  - Foreign keys: ExpertId, SpecializationId. |
| 17 | ExpertTimeSlots | Stores expert availability slots used for scheduled consultation booking.  - Primary key: Id.  - Foreign keys: ExpertId. |
| 18 | FirstAidGuidelines | Stores first-aid guideline content used by the venom and snake knowledge domain.  - Primary key: Id.  - Foreign keys: none. |
| 19 | GeographicRegion | Stores geographic region master data used for distribution and mapping features.  - Primary key: Id.  - Foreign keys: none. |
| 20 | Lessons | Stores educational lesson content with category and publication status for the learning feature.  - Primary key: Id.  - Foreign keys: none. |
| 21 | LibraryMedias | Stores media assets associated with snake species in the snake library.  - Primary key: Id.  - Foreign keys: SnakeSpeciesId, UploadedById. |
| 22 | LocationEvents | Stores time-stamped location tracking events for a session, including the account, role, coordinates, speed, and heading.  - Primary key: Id.  - Foreign keys: no explicit FK is modeled in the ERD, but current code uses SessionId and AccountId as logical references. |
| 23 | MemberProfiles | Stores member-specific profile information such as rating, emergency contacts, and health indicators.  - Primary key: AccountId.  - Foreign keys: AccountId. |
| 24 | PaymentCards | Stores payment card details and default-card status for payment-related features.  - Primary key: Id.  - Foreign keys: none explicitly modeled in the ERD or current entity. |
| 25 | RegionSnakeMapping | Stores the relationship between snake species and geographic regions, including distribution priority and commonness.  - Primary key: Id.  - Foreign keys: SnakeSpeciesId, GeographicRegionId. |
| 26 | ReportMedias | Stores uploaded media files attached to multiple business entities through a polymorphic ReferenceId and ReferenceType pattern, with processing and sequencing metadata.  - Primary key: Id.  - Foreign keys: no explicit FK is modeled in the final design because parent linkage is polymorphic. |
| 27 | RescueMissions | Stores rescue mission execution records created to handle snakebite incidents.  - Primary key: Id.  - Foreign keys: RescuerId, IncidentId. |
| 28 | RescueRequestSessions | Stores dispatch sessions that group rescuer matching attempts for a snakebite incident.  - Primary key: Id.  - Foreign keys: IncidentId. |
| 29 | RescuerProfiles | Stores rescuer-specific profile information such as online status, rating, location, and mission counters.  - Primary key: AccountId.  - Foreign keys: AccountId. |
| 30 | RescuerRequests | Stores the request and response record for each rescuer contacted during a rescue dispatch session.  - Primary key: Id.  - Foreign keys: SessionId, IncidentId, RescuerId. |
| 31 | ShiftAssignment | Stores the assignment of rescuers to work shifts together with check-in, check-out, and activity status.  - Primary key: Id.  - Foreign keys: RescuerId, ShiftId. |
| 32 | SnakeAIRecognitionResults | Stores AI snake recognition outputs for uploaded report media, including mapped species, confidence, review status, and expert - correction data.  - Primary key: Id.  - Foreign keys: ReportMediaId, AIModelId, DetectedSpeciesId, ExpertCorrectedSpeciesId, ExpertId, SnakeCatchingMissionId. |
| 33 | SnakebiteIncidents | Stores snakebite emergency incident records, including reporter, assigned rescuer, location, symptoms, AI/species identification, and dispatch status.  - Primary key: Id.  - Foreign keys: UserId, AssignedRescuerId, AIRecognitionResultId, IdentifiedSnakeSpeciesId. |
| 34 | SnakeCatchingMissions | Stores snake catching mission execution records that fulfill snake catching requests.  - Primary key: Id.  - Foreign keys: RescuerId, SnakeCatchingRequestId, CatchingEnvironmentId. |
| 35 | SnakeCatchingRequests | Stores snake catching requests created by users, including location, priority, assignment, timing, and pricing information.  - Primary key: Id.  - Foreign keys: UserId, AssignedRescuerId. |
| 36 | SnakeSpecies | Stores the main snake species master data used by the library, AI mapping, identification, and medical guidance features.  - Primary key: Id.  - Foreign keys: none. |
| 37 | SnakeSpeciesNames | Stores alternative names and slugs for a snake species to support lookup and multilingual naming.  - Primary key: Id.  - Foreign keys: SnakeSpeciesId. |
| 38 | Specializations | Stores the master list of expert specializations.  - Primary key: Id.  - Foreign keys: none. |
| 39 | SpeciesAntivenoms | Stores the many-to-many relationship between snake species and antivenoms.  - Primary key: Id.  - Foreign keys: SnakeSpeciesId, AntivenomId. |
| 40 | SpeciesVenoms | Stores the many-to-many relationship between snake species and venom types.  - Primary key: SnakeSpeciesId, VenomTypeId.  - Foreign keys: SnakeSpeciesId, VenomTypeId. |
| 41 | SymptomConfigs | Stores symptom configuration, scoring, alerts, and display metadata for each venom type.  - Primary key: Id.  - Foreign keys: VenomTypeId. |
| 42 | SystemSettings | Stores configurable system settings as key-value pairs with type metadata.  - Primary key: SettingKey.  - Foreign keys: none. |
| 43 | TrackingSessions | Stores the current live-tracking state for a mission or request session, including participant locations, freshness timestamps, and calculated distance/ETA.  - Primary key: Id.  - Foreign keys: no explicit FK is modeled in the ERD because SessionId is used polymorphically with SessionType. |
| 44 | Transactions | Stores monetary transaction records for users, including amount, type, payment method, gateway reference, and business reference. - - Primary key: Id.  - Foreign keys: UserId is optional in current code; ReferenceId is a logical business reference rather than an explicit FK. |
| 45 | TreatmentFacilities | Stores treatment facility master data such as name, address, contact, and location.  - Primary key: Id.  - Foreign keys: none. |
| 46 | UserFeedbacks | Stores ratings and comments created by one user for another user in the context of a referenced business record such as a consultation or mission.  - Primary key: id.  - Foreign keys: RaterId, TargetUserId; ReferenceId is a logical reference used together with Type. |
| 47 | VenomTypes | Stores venom type master data and links each venom type to its first-aid guideline.  - Primary key: Id.  - Foreign keys: FirstAidGuidelineId. |
| 48 | Wallets | Stores each user wallet and its current balance.  - Primary key: Id.  - Foreign keys: UserId. |
| 49 | WalletWithdraws | Stores withdrawal requests created against user wallets, including bank information, amount, status, and processing outcome.  - Primary key: Id.  - Foreign keys: UserId, WalletId. |
| 50 | WorkShift | Stores shift-definition master data, including time range, required rescuers, and active status.  - Primary key: Id.  - Foreign keys: none. |

###### Figure 4.2.2 Database Design Description

## 3. Detailed Design

### 3.1 Snakebite Incident Emergency

#### 3.1.1 Class Diagram

![](data:image/png;base64...)

###### Figure 4.3.1.1.1 Snakebite Incident Class Diagram

#### 3.1.2 Sequence Diagram Create SnakeBite Incident![](data:image/png;base64...)

###### Figure 4.3.1.2.1 Create Snakebite Incident Sequence Diagram

#### 3.1.3 Sequence Diagram Verify Snakebite Incident

![](data:image/png;base64...)

###### Figure 4.3.1.3.1 Verify Snakebite Incident Sequence Diagram

#### 3.1.4 Sequence Diagram False Alarm Snakebite Incident

![](data:image/png;base64...)

###### Figure 4.3.1.4.1 False Alarm Snakebite Incident Sequence Diagram

#### 3.1.5 Sequence Diagram Dispatch Rescue Request

![](data:image/png;base64...)

###### Figure 4.3.1.5.1 Dispatch Rescue Request Sequence Diagram

#### 3.1.6 Sequence Diagram Accepts Rescue Request

![](data:image/png;base64...)

###### Figure 4.3.1.6.1 Accept Rescue Request Sequence Diagram

#### 3.1.7 Sequence Diagram Decline Rescue Request

![](data:image/png;base64...)

###### Figure 4.3.1.7.1 Decline Rescue Request Sequence Diagram

#### 3.1.8 Sequence Diagram Mark Rescue Mission Start

![](data:image/png;base64...)

###### Figure 4.3.1.8.1 Start Rescue Mission Sequence Diagram

#### 3.1.9 Sequence Diagram Mark Rescue Mission Arrived

![](data:image/png;base64...)

###### Figure 4.3.1.9.1 Mark Rescue Mission Arrived Sequence Diagram

#### 3.1.10 Sequence Diagram Mark Rescue Mission Complete

![](data:image/png;base64...)

###### Figure 4.3.1.10.1 Complete Rescue Mission Sequence Diagram

#### 3.1.11 Sequence Diagram Cancel Snakebite Incident

![](data:image/png;base64...)

###### Figure 4.3.1.11.1 Complete Rescue Mission Sequence Diagram

#### 3.1.12 Sequence Diagram Payment Snakebite Incident Wallet

![](data:image/png;base64...)

###### Figure 4.3.1.12.1 Payment Snakebite Incident by Wallet Sequence Diagram

#### 3.1.13 Sequence Diagram Create Snakebite Incident PayOS Payment Link

***![](data:image/png;base64...)***

###### Figure 4.3.1.13.1 Create Snakebite Incident PayOS Payment Link Sequence Diagram

#### 3.1.14 Sequence Diagram Payment Snakebite Incident via PayOS

***![](data:image/png;base64...)***

###### Figure 4.3.1.14.1 Payment Snakebite Incident via PayOS Sequence Diagram

###

### 3.2 Snake Capture

#### 3.2.1 Class Diagram

![](data:image/png;base64...)

###### Figure 4.3.2.1.1 Snake Capture Class Diagram

#### 3.2.2 Sequence Diagram Create new Snake Catching Request

###### Figure 4.3.2.2.1 Create Snake Catching Request Sequence Diagram![](data:image/png;base64...)

#### 3.2.3 Sequence Diagram Confirm Snake Catching Request![](data:image/png;base64...)

###### Figure 4.3.2.3.1 Confirm Snake Catching Request Sequence Diagram

####

#### 3.2.4 Sequence Diagram Assign Snake Catching Request

###### Figure 4.3.2.4.1 Assign Snake Catching Request Sequence Diagram![](data:image/png;base64...)

#### 3.2.5 Sequence Diagram Cancel Snake Catching Request![](data:image/png;base64...)

###### Figure 4.3.2.5.1 Cancel Snake Catching Request Sequence Diagram

####

#### 3.2.6 Sequence Diagram View List Snake Catching Request

####

#### ![](data:image/png;base64...)

###### Figure 4.3.2.6.1 View List Catching Request Sequence Diagram

####

#### 3.2.7 Sequence Diagram Start Snake Catching Mission

###### Figure 4.3.2.7.1 Start Snake Catching Mission Sequence Diagram![](data:image/png;base64...)

#### 3.2.8 Sequence Diagram Mark as Arrived Snake Catching Mission

###### Figure 4.3.2.8.1 Mark Arrived Snake Catching Mission Sequence Diagram![](data:image/png;base64...)

#### 3.2.9 Sequence Diagram Complete Snake Catching Mission

###### Figure 4.3.2.9.1 Complete Snake Catching Mission Sequence Diagram![](data:image/png;base64...)

###

###

###

###

###

###

###

### 3.3 Consultation

* Consultation module supports two primary business modes:
* Scheduled Consultation: user books a future time slot, pays, joins room at slot time, ends consultation, and leaves review.
* Emergency Consultation: user creates instant request, pays first, waits for expert accept/reject, then joins room if accepted.
* The module also includes in-room real-time capabilities (chat, attachment, signal events), LiveKit token generation, and settlement lifecycle handling.
* Canonical business states used in this feature:
* Booking: PendingPayment, Confirmed, Completed.
* Emergency Request: PendingPayment, PendingExpertResponse, AcceptedByExpert, DeclinedByExpert, Expired.

#### 3.3.1 Class Diagram

###### ![](data:image/png;base64...)

######

###### Figure 4.3.3.1.1 Consultation Class Diagram

####

####

#### 3.3.2 Sequence Diagram View List Experts and Presence

**![](data:image/png;base64...)**

###### Figure 4.3.3.2.1 View List Expert and Presence Class Diagram

#### 3.3.3 Sequence Diagram Create and Pay Scheduled Booking

![](data:image/png;base64...)

###### Figure 4.3.3.3.1 Create and Pay Scheduled Booking Class Diagram

#### 3.3.4 Sequence Diagram Create, Pay, and Notify Emergency Consultation Request

![](data:image/png;base64...)

###### Figure 4.3.3.4.1 Create, Pay, Notify Emergency Consultation Request Class Diagram

#### 3.3.5 Sequence Diagram Expert Accept/Reject Emergency Request

**![](data:image/png;base64...)**

###### Figure 4.3.3.5.1 Expert Accept or Reject Emergency Consultation Request Class Diagram

#### 3.3.6 Sequence Diagram Join Consultation Room and In-Room Interaction

**![](data:image/png;base64...)**

###### Figure 4.3.3.6.1 Join Consultation Room and In-Room Class Diagram

#### 3.3.7 Sequence Diagram End Consultation and Settlement

###### **![](data:image/png;base64...)** Figure 4.3.3.7.1 End Consultation Class Diagram

#### 3.3.8 Sequence Diagram Create Consultation Review

**![](data:image/png;base64...)**

###### Figure 4.3.3.8.1 Create Consultation Review Class Diagram

# V. Software Testing Documentation

## 1. Scope of Testing

| **Level** | **In-Charge** | **Inputs/Time** | **Focus** | **Acceptance Criteria** |
| --- | --- | --- | --- | --- |
| Unit Testing | Developers | Individual modules during development. | Validate logic, functions, edge cases. | All unit tests pass, no critical bugs. |
| Integration Testing | All Members | After module integration. | API communication, data flow, service interaction. | Modules integrate correctly, no major errors. |
| Interface Testing | All Members | After UI implementation. | UI behavior, navigation, input validation. | UI works correctly, no broken navigation. |
| End-to-End Testing | All Members | Before release, full system testing. | Complete user workflows. | Main user flows work, system ready for release. |

###### Table 5.1.1 Scope of Testing

##

## 2. Test Strategy

### 2.1 Testing Types

| **Objective** | **Technique** | **Completion Criteria** |
| --- | --- | --- |
| Testing the verification and validation of the units of code. | Unit testing | All unit tests pass, no critical logic errors. |
| Testing the group of integration modules. | Integration testing | Modules integrate correctly, no major integration issues. |
| Testing the communication between components or systems by data and control. | Interface testing | Interface works correctly, no broken data flow. |
| Testing application environments in real situations. | End-to-end testing | Main user flows work successfully, system ready for release. |

###### Table 5.2.1.1 Testing Types

### 2.2 Test Levels

| **Type of Tests** | **Test Level** | | | |
| --- | --- | --- | --- | --- |
| **Unit** | **Integration** | **System** | **Acceptance** |
| Unit testing | X |  |  |  |
| Integration testing |  | X |  |  |
| Interface testing |  | X | X |  |
| End-to-end testing |  |  | X | X |

###### Table 5.2.2.1 Test Levels

### 2.3 Supporting Tools

| **Purpose** | **Tool** | **Vendor/In-house** | **Version** |
| --- | --- | --- | --- |
| API testing | Postman/Swagger | Postman/Swagger | latest |

###### Table 5.2.3.1 Supporting Tools

##

## 3. Test Plan

### 3.1 Human Resources

| **Worker/Doer** | **Role** | **Specific Responsibilities/Comments** |
| --- | --- | --- |
| Đoàn Ngọc Trung | Leader | Planning, assign feature groups to test, write tests and test reports. |
| Nguyễn Văn Duy Khiêm | Member | Planning, assign feature groups to test, write tests and test reports. |
| Nguyễn Mạnh Dưỡng | Member | Write tests and test reports. |
| Nguyễn Phúc Nhân | Member | Write tests and test reports. |
| Phan Anh Khoa | Member | Write tests and test reports. |

###### Table 5.3.1.1 Human Resources

### 3.2 Test Environment

| **Purpose** | **Tool** | **Provider** | **Version** |
| --- | --- | --- | --- |
| Testing API Tool | Windows | Microsoft | 11 |

###### Table 5.3.2.1 Test Environment

### 3.3 Test Milestones

| **Milestone Task** | **Start Date** | **End Date** |
| --- | --- | --- |
| Unit Test | 2026/03/02 | 2026/04/16 |
| Other Test Case (IT, ST, AT) | 2026/01/20 | 2026/04/12 |

###### Table 5.3.3.1 Test Milestones

##

## 4. Test Cases

### 4.1 Unit Test Cases

[SnakeAid Report5\_Unit Test.xlsx](https://docs.google.com/spreadsheets/d/1xjUImFgvcjLX8f_t3mVEedaKhp11-QQJ/edit?gid=1022734313#gid=1022734313)

![](data:image/png;base64...)

###### Figure 5.4.1.1 Unit Test Cases

###

###

###

###

###

### 4.2 Other Test Cases (IT,ST,AT)

[Report5\_Test\_Report.xlsx](https://docs.google.com/spreadsheets/d/1bYQmSZ15vVAKlwqXVUUVSSgcm5OcKP2k/edit?gid=1977667783#gid=1977667783)

**![](data:image/png;base64...)**

###### Figure 5.4.2.1 Other Test Cases

##

## 5. Test Reports

### 5.1 Unit Test Report

[SnakeAid Report5\_Unit Test.xlsx](https://docs.google.com/spreadsheets/d/1xjUImFgvcjLX8f_t3mVEedaKhp11-QQJ/edit?gid=1022734313#gid=1022734313)

![](data:image/png;base64...)

###### Figure 5.5.1.1 Unit Test Cases Statistic

## 5.2 Other Test Report (IT,ST,AT)

[Report5\_Test\_Report.xlsx](https://docs.google.com/spreadsheets/d/1bYQmSZ15vVAKlwqXVUUVSSgcm5OcKP2k/edit?gid=1977667783#gid=1977667783)

**![](data:image/png;base64...)**

###### Figure 5.5.2.1 Other Test Cases Statistic

#

#

#

#

# VI. Release Package & User Guides

## 1. Deliverable Package

| **No.** | **Deliverable Item** | **Description** |
| --- | --- | --- |
| 1 | Project Schedule/Tracking | - Zalo  - Discord - Google Sheet - Google Docs |
| 2 | Source Codes | - Version Control: Git  - Source Code Management: GitHub |
| 3 | Database Script(s) | - Stored at EF core migration classes |
| 4 | Final Report Document | - Google Docs  - Google Drive |
| 5 | Test Cases Document | - Google Sheet |
| 6 | Slide | - Canva |
| 7 | Defects List | - Google Sheet |
| 8 | Issues List | - Google Sheet |

###### Table 6.1.1 Deliverable Package

## 2. Installation Guides

### 2.1 System Requirements

|  | **Minimum** | **Recommended** |
| --- | --- | --- |
| Operating System | Windows/macOS/Linux 64-bit, Android 7.0+ (API 24) | Windows/macOS/Linux 64-bit, Android 10+ |
| CPU | 2 cores | 4 cores |
| RAM | 4GB | 16GB |
| Network Connection | Internet access (Wi-Fi or mobile data) | Stable Wi-Fi or 4G/5G connection |
| Camera and Microphone | Required for video consultation and image capture | Higher-quality front and rear cameras with clear microphone input |
| Location Services | GPS/location permission required | High-accuracy location enabled |
| Notifications | Push notifications supported | Notifications enabled at all times |
| Mobile Storage | At least 500 MB available | 1 GB or more available |

###### Table 6.2.1.1 System Requirements

### 2.2 Installation Instruction

#### 2.2.1 Frontend

##### **2.2.1.1 Frontend Mobile Application**

This is the installation guide for the Frontend Mobile project of SnakeAid.

* **Step 1:** Install required tools.
  + Flutter SDK (stable, recommended: 3.41.1)
  + JDK 17 (recommended for Android builds)
  + Android Studio (Android SDK + emulator)
  + Git
* **Step 2:** Run git clone command on the frontend repository.
  + git clone [*https://github.com/Snake-AID/SnakeAid.Mobile.git*](https://github.com/Snake-AID/SnakeAid.Mobile.git)
* **Step 3:** Navigate to the SnakeAid.Mobile directory.
* **Step 4:** Create a **.env** file based on **.env.example** and configure the following values.

# API Configuration

BASE\_URL=https://dev.snakeaid.tech

# BASE\_URL=http://192.168.2.17:8000

API\_TIMEOUT=30000

# Google Maps API Key

GOOGLE\_MAPS\_API\_KEY=YOUR\_GOOGLE\_MAPS\_API\_KEY\_HERE

# OpenRouteService API Key (for routing)

# Get FREE API key at: https://openrouteservice.org/dev/#/signup

OPENROUTE\_API\_KEY=YOUR\_OPENROUTE\_API\_KEY\_HERE

# Firebase Configuration (optional - for push notifications)

# FIREBASE\_API\_KEY=your\_firebase\_api\_key

# FIREBASE\_PROJECT\_ID=your\_project\_id

# FIREBASE\_MESSAGING\_SENDER\_ID=your\_sender\_id

# FIREBASE\_APP\_ID=your\_app\_id

# App Configuration

APP\_NAME=SnakeAid

APP\_VERSION=1.0.0

ENVIRONMENT=development

* **Step 5:** Create (or select) a Firebase project and register the existing Android application.
  + Open Firebase Console: <https://console.firebase.google.com/>
  + Create a new Firebase project (or select an existing one).
  + In Project settings, go to Your apps and click Add app > Android.
  + Enter the Android package name from applicationId in android/app/build.gradle.kts.
  + Complete app registration in Firebase.
* **Step 6:** Download google-services.json and place it in the Android app folder.

*![](data:image/png;base64...)*

* Step 7: Run flutter pub get to install all dependencies.

*![](data:image/png;base64...)*

###### Figure 6.2.2.1.1.1 Flutter Pub Get Result

* **Step 8:** Run **flutter run** to launch the mobile application.
* **Step 9:**Confirm the mobile application is up and running on emulator or physical device.

![](data:image/png;base64...)

###### Figure 6.2.2.1.1.2 Role Selection Screen

##### **2.2.1.2 Frontend Web Application**

This is the installation guide for the Frontend Web project of SnakeAid.

* **Step 1:** Install required tools.
  + Node.js (minimum: 20.x, recommended: 22.x LTS)
  + npm (comes with Node.js)
  + Git
  + Visual Studio Code
* **Step 2:** Run git clone command on the frontend repository.
  + git clone <https://github.com/Snake-AID/SnakeAid.Frontend.git>
* **Step 3:** Navigate to the SnakeAid.Frontend directory.
* **Step 4:** Create a **.env.local** file in the project root and configure the following values.

NEXT\_PUBLIC\_API\_BASE\_URL=https://dev.snakeaid.tech/

# NEXT\_PUBLIC\_API\_BASE\_URL=http://localhost:3000/

1. **Step 5:** Run **npm install** to install all dependencies.

![](data:image/png;base64...)

###### Figure 6.2.2.1.2.1 Npm Install Result

* **Step 6:** Run **npm run build** to compile the Next.js application (mandatory step).

*![](data:image/png;base64...)*

###### Figure 6.2.2.1.2.2 Npm Run Build Result

* **Step 7:** Run **npm run dev** to launch the Next.js development server.
* **Step 8:** Open *http://localhost:3000* in a browser and confirm the web application is up and running.

![](data:image/png;base64...)

###### Figure 6.2.2.1.2.3 Web Login Screen

#### 2.2.2 Backend

* **Step 1:** Install **Git.**
* **Step 2:** Connect to **GitHub** service.
* **Step 3:** Create a directory to store all backend source code.
* **Step 4:** Run **git clone** command on **GitHub** repository in the created directory:
  + <https://github.com/Snake-AID/SnakeAid.Backend.git>
* **Step 5:** Run **docker pull** command to pull and set up the RabbitMQ image.
  + docker run -d \

--hostname rabbitmq-host \

--name rabbitmq \

-e RABBITMQ\_DEFAULT\_USER=admin \

-e RABBITMQ\_DEFAULT\_PASS=admin123 \

-p 5672:5672 \

-p 15672:15672 \

rabbitmq:3-management

* **Step 6:** Get **Firebase Service Account Key** for Server
  + Open **Firebase Console:** <https://console.firebase.google.com>
  + Select your **Firebase** project.
  + In the left sidebar, click **Project settings.**
  + Click the **Service accounts** tab.
  + Under **Firebase Admin SDK**, click **Generate new private key**.
  + Confirm the action - a JSON file will be downloaded to your computer.
  + Placed this file into SnakeAid.Api project, same level with Program.cs

![](data:image/png;base64...)

* + Never commit this JSON file to public repositories. Treat it like a password.
* **Step 7:** Setup the env for project:

#### 2.2.3 Ai Inference Server

* **Step 1: Install required tools.**
  + Docker (recommended latest version)
  + Git
* **Step 2: Clone the AI inference server repository.**
  + git clone <https://github.com/Snake-AID/SnakeAI-Model-Endpoint.git>
* **Step 3: Navigate to the project directory.**
  + cd SnakeAI-Model-Endpoint
* **Step 4: Prepare model and data directories.**
  + Create the following folders in the project root:
  + mkdir -p models
  + mkdir -p data/saved\_images
  + Place your trained YOLO model file (best.pt) into:
  + /models/best.pt
* **Step 5: Build the Docker image.**
  + docker build -t snake-detect:latest .
* **Step 6: Run the Docker container.**

**![](data:image/png;base64...)**

* **Step 7: Verify the server is running.**
  + Open browser: http://localhost:8000/health
  + Expected response:

![](data:image/png;base64...)

* **Step 8: Access API documentation (Swagger UI).**
  + Open:
  + http://localhost:8000/swagger
  + This interface allows testing all endpoints directly.
* **Step 9: Test inference endpoints.**
* Option 1: Upload image file

![](data:image/png;base64...)

* Option 2: Base64 image

![](data:image/png;base64...)

* Option 3: Image URL

![](data:image/png;base64...)

* **Step 10: Confirm inference response.**
  + Example response:

![](data:image/png;base64...)

## 3. User Manual

### 3.1 Overview

SnakeAid is a comprehensive snake emergency support system designed to assist users in snake-related incidents, including snake bites, snake catching requests, and expert consultations. The system connects multiple roles including Members, Operators, Rescuers, Experts, and Administrators to ensure fast response, effective coordination, and safe handling of snake-related emergencies.

The system supports multiple service workflows, each designed for specific user needs such as emergency rescue, snake catching, consultation services, and system management.

The following workflows are available in the SnakeAid system:

**Workflow 1: SOS Snake Bite**

This workflow handles emergency snake bite cases requiring immediate assistance.

* Member opens the SnakeAid application and triggers the SOS emergency
* SnakeAid System creates an Emergency Case with status Pending and notifies the Operator
* Member submits emergency information including GPS location, symptoms, and optional snake image
* Operator verifies the emergency case with the Member and updates status to Verified
* Operator assigns an available Rescuer and updates status to Assigned
* SnakeAid System sends real-time assignment notification to the Rescuer and updates Member with Rescuer information and estimated arrival time (ETA)
* Rescuer accepts the mission and travels to the Member's location
* Rescuer arrives at the scene and provides first aid support
* Rescuer completes the emergency support and submits mission result with evidence
* SnakeAid System updates case status to Finished and sends payment request to Member
* Member completes payment via available payment methods
* SnakeAid System marks the case as Completed, settles payment, and stores case history

**Workflow 2: Snake Catching**

This workflow allows Members to request snake catching services.

* Member creates a snake catching request (photo/species, location, address detail).
* SnakeAid System creates requests as Pending, calculates estimated price and distance, and notifies the Operator.
* Members can pay travel fees in allowed states.
* Operator verifies request with Member and updates status to Confirmed.
* Operator assigns a Rescuer and updates status to Assigned.
* SnakeAid System sends SignalR assignment to Rescuer and assignment notification to Member.
* Rescuer executes mission flow (Preparing, EnRoute, Arrived), captures snake, and uploads evidence.
* SnakeAid System marks request Finished and calculates actual cost.
* Member reviews service details and pays Round 2 service fee.
* SnakeAid System marks request Completed, sends completion notifications, and stores history.

**Workflow 3: Scheduled Consultation**

This workflow allows Members to schedule consultation sessions with Experts.

* Member opens the Expert directory and selects an Expert.
* Member taps "Chọn Đặt Lịch" on the service selection screen, then selects a date and time slot.
* SnakeAid System validates slot availability.
* Member enters consultation details and confirms "Xác Nhận Đặt Lịch".
* SnakeAid System creates a scheduled booking (pending payment) and locks the selected slot.
* Member confirms payment for the consultation fee on "Xác Nhận & Thanh Toán".
* Payment gateway processes the transaction and holds escrow.
* Member is redirected to consultation home with the new consultation highlighted.
* At consultation time, Member and Expert join the waiting room and then enter the video consultation session.
* When the session is ended, SnakeAid System marks consultation completion, stores consultation history, and supports rating flow.

**Workflow 4: Instant Consultation**

This workflow supports real-time consultation with available Experts.

* Member taps "Chọn Tư Vấn Ngay" on an available Expert service selection screen.
* SnakeAid System validates whether the Expert is online.
* If the Expert is unavailable, Member is prompted to choose another Expert or switch to scheduled flow.
* Member enters consultation details and confirms payment on "Xác Nhận & Thanh Toán".
* SnakeAid System creates an emergency request and processes payment (Wallet or PayOS).
* Payment gateway holds escrow and returns payment status.
* SnakeAid System opens the emergency waiting status (PendingExpertResponse) and notifies both sides through realtime updates.
* Expert can accept or reject the request from Expert Home.
* If accepted, both parties enter waiting room and then video consultation.
* Session ends, completion is recorded, escrow is settled by backend policy, and Member can submit rating/feedback.

**Workflow 5: System Administration**

This workflow describes administrative management features.

* Admin starts from Admin Dashboard after successful sign-in.
* Admin can monitor analytics widgets and overall operational status from "Bảng điều khiển".
* Admin can manage user accounts in "Quản lý người dùng", including search/filter, detail review, and lock/unlock actions.
* Admin can manage staff scheduling in "Quản lý lịch làm việc", including shift templates and rescuer assignments.
* Admin can supervise request operations through "Quản lý sự cố", "Quản lý yêu cầu bắt rắn", and "Quản lý phiên tư vấn".
* Admin can maintain domain data in "Quản lý loài rắn", "Quản lý huyết thanh", and "Quản lý cơ sở điều trị".
* Admin can control financial operations in "Quản lý giao dịch" and "Duyệt rút tiền".
* Admin can govern content in "Quản lý bài học" and "Quản lý bài viết" with create/update/delete and moderation flows.
* Admin can review AI recognition records in "Quản lý ảnh báo cáo rắn" with filtering and detail inspection.
* Admin can manage runtime configuration in "Quản lý cấu hình động" and apply reload actions.

### 3.2 SOS Snake Bite

**Purpose**

This workflow allows Members to request emergency snake bite assistance and receive immediate support from rescuers.

**Step-by-Step Guide**

**Step 1.1: Hold SOS Emergency Button**

Member opens SnakeAid application and holds the SOS Emergency button from the home screen.

**Steps:**

1. Open SnakeAid Application.
2. Hold the SOSEmergency button for 2 seconds.

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.1 Member Home Screen — SOS Button

Step 1.2: Emergency Tracking Screen Activated

After holding the SOS button, the system navigates to the SOS emergency screen. The system automatically:

* Detects GPS location
* Displays map view
* Searches nearby rescuers
* Shows emergency quick actions

Member can perform additional optional actions to support emergency handling.

![](data:image/png;base64...)

###### Figure 6.3.2.2 Emergency Tracking Screen

**Step 1.3 Optional Emergency Actions**

Member can perform the following actions. These actions are optional and do not require sequential order.
Member can perform any action at any time depending on the emergency situation.

The system provides the following emergency support options:

* Snake Identification
* Report Symptoms
* First Aid
* Severity Level
* Case Details

**Option 1: Snake Identification**

Member can identify snake species to receive appropriate first aid instructions. The system provides AI-powered snake recognition and alternative manual selection.

![](data:image/png;base64...)

###### Figure 6.3.2.3 Snake Identification Screen

**Option 1.1: Identify Snake Using Image (Recommended)**

Member captures a snake image for AI-based identification.

Steps:

1. Tap “Nhận dạng” button
2. Capture snake image using camera
3. Confirm captured image
4. Submit image for AI recognition
5. System processes image
6. System displays snake species identification
7. Member taps "Xem hướng dẫn sơ cứu"
8. System displays first aid instructions for detected snake

**System Actions:**

* AI identifies snake species
* System evaluates danger level
* First aid instructions displayed
* Information sent to Operator and Rescuer

**Expected Result:**

* Snake species detected
* First aid instructions displayed
* Severity level evaluated

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.4 Snake Confirmation Screen

**Option 1.2: Select Snake Species Near Your Area**

If member does not have snake image, member can manually select snake species near their location.

**Steps:**

1. Tap “Tôi không có ảnh rắn**”**
2. System navigates to **Snake Selection by Location** Screen
3. System displays list of snake species near user location
4. Member selects suspected snake species
5. System displays first aid step for this snake

**System Actions:**

* System filters snakes by location
* System displays common snakes in nearby area
* First aid instructions displayed

**Expected Result:**

* Snake species selected
* First aid instructions displayed
* Rescuer receives updated information

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.5 Snake Selection by Location Screen

**Option 2: Report Symptoms**

Member can report symptoms to help the system evaluate the severity of the snake bite and prioritize emergency response.

![](data:image/png;base64...)

###### Figure 6.3.2.6 Symptom Report Screen

**Steps:**

1. Tap “Triệu chứng” button
2. System navigates to Report Symptoms Screen
3. Member selects symptoms from available list
4. Member confirms selected symptoms
5. Tap “Xác nhận”
6. System evaluates severity level
7. System displays emergency severity result

**System Actions**

* System analyzes reported symptoms
* System evaluates severity level (Low / Medium / High / Critical)
* System prioritizes rescuer dispatch
* System updates Operator and Rescuer information

**Expected Result**

* Severity level displayed
* Rescuer receives additional emergency information
* Emergency case priority updated

**Notes**

* Reporting symptoms is optional
* Member can update symptoms anytime
* Severity evaluation helps rescuer prepare equipment

**Option 3: First Aid**

Member can view first aid instructions based on snake identification results.

If the snake species has been identified, the system displays specific first aid instructions.
If no snake species is identified, the system displays general snake bite first aid instructions.

**Steps**

1. Member taps **"**Sơ cứu**"** button
2. System checks snake identification result

* Case 1: Snake identified
   System displays specific first aid instructions
* Case 2: Snake not identified
   System displays general snake bite first aid instructions

**System Actions**

* System checks snake identification data
* System loads corresponding first aid instructions
* System displays first aid steps

**Expected Result**

* Member receives first aid instructions
* Member performs immediate first aid

![](data:image/png;base64...)

###### Figure 6.3.2.7 First Aid Steps Screen (Snake Species Identified)

![](data:image/png;base64...)

###### Figure 6.3.2.8 First Aid Steps Screen (General First Aid)

**Option 4: Severity Level**

Member can view the emergency severity level based on reported symptoms.

If symptoms are reported, severity level will be displayed.

If no symptoms are reported, severity level will not be available.

**Steps**

1. Member taps **"**Mức độ**"** button
2. System checks symptom report

* Case 1: Symptoms reported: system displays severity level
* Case 2: No symptoms reported

**System Actions**

* System checks symptom data
* System evaluates severity level
* System displays severity result

**Expected Result**

* Severity level displayed
* Member understands emergency seriousness

![](data:image/png;base64...)

###### Figure 6.3.2.9 Severity Assessment Screen

**Option 5: Case Details**

Member can view current emergency case information and status.

**Steps**

* Member taps "Chi tiết" button
* System navigates to Case Details Screen
* System displays emergency case information

**Display Information**

* Case ID
* Emergency Status
* Member Location
* Assigned Rescuer (if available)
* Snake Identification (if available)
* Symptoms Report (if available)
* Severity Level (if available)

**System Actions**

* System retrieves emergency case information
* System displays real-time status

**Expected Result**

* Member understands current emergency status
* Member tracks emergency progress

![](data:image/png;base64...)

###### Figure 6.3.2.10 Case Details Screen

**Step 1.4: Cancel Emergency Request (Optional)**

* Member can cancel the emergency request if the situation is resolved or the request was created by mistake.
* This action is optional and available before the rescuer arrives.

**Steps**

* Member taps "Hủy yêu cầu" button
   System displays confirmation dialog
   Member confirms cancellation
   System cancels emergency request
   System updates case status to Cancelled

**System Actions**

* System updates emergency status to Cancelled
* System notifies Operator
* System notifies assigned Rescuer (if available)
* System stops rescuer tracking

**Expected Result**

* Emergency request cancelled
* Operator notified
* Rescuer notified
* Case closed

![](data:image/png;base64...)

###### Figure 6.3.2.11 Cancel Emergency Confirmation Dialog

![](data:image/png;base64...)

###### Figure 6.3.2.12 Activity Detail - With Cancelled Status (Member)

**Step 2: Operator Verifies Emergency Case**

After an SOS request is created, the system notifies the Operator to verify the emergency case.

The Operator reviews the emergency information and contacts the Member to confirm the situation.

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.13 Operator Dashboard

**System Actions**

* System creates Emergency Case
* Status = **Pending**
* System notifies Operator
* Operator receives SOS alert

**Operator Actions**

1. Login role Operator on web
2. Operator opens Emergency Case Management
3. Operator reviews SOS information
4. Operator checks:

* Member location
* Snake identification (if available)
* Reported symptoms (if available)

1. Operator contacts Member (if necessary)
2. Operator verifies emergency case
3. Operator click “Xác minh” to updates status “Verified”

**Expected Result**

* Emergency case verified
* Case ready for rescuer assignment

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.14 Emergency Case Detail — Pending Status (Operator View)

**Step 3: Operator Assigns Rescuer**

After verifying the case, the Operator assigns a Rescuer to handle the emergency.

![](data:image/png;base64...)

###### Figure 6.3.2.15 Emergency Case Detail — Verified Status (Operator View)

**Steps**

1. Operator selects **“**Điều phối đội cứu hộ**”** button
2. System displays available rescuers nearby
3. Operator selects suitable rescuer
4. Operator confirms assignment
5. System updates case status to “Assigned”

**System Actions**

* System sends notification to Rescuer
* System sends ETA to Member
* System updates emergency status

**Expected Result**

* Rescuer assigned
* Member receives rescuer information
* Rescuer receives emergency request

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.16 Assign Rescuer Screen

![](data:image/png;base64...)

###### Figure 6.3.2.17 Emergency Case Detail — Assigned Status (Operator View)

**Step 3.1: Rescuer Assignment Expired (Alternative Flow)**

If the rescuer does not accept the assignment within the allowed time, the request will automatically expire.

**Trigger Condition**

* Rescuer does not tap “nhận nhiệm vụ”within the allowed time
* Assignment request expires automatically

**System Actions**

* System updates assignment status to Expired
* System displays notification to Operator
* System shows message:

**"**The rescuer did not respond. Assignment expired.**"**

* System allows Operator to assign another rescuer

**Operator Actions**

1. Operator receives expired notification
2. Operator opens emergency case
3. Operator clicks “Điều phối đội cứu hộ” button
4. Operator selects another rescuer
5. Operator taps “Xác nhận điều phối” button

**Expected Result**

* New rescuer assigned
* Emergency case continues

![](data:image/png;base64...)

###### Figure 6.3.2.18 Assignment Expired Notification (Operator View)

**Step 4: Rescuer Accepts Emergency Request**

Rescuer receives an SOS request and accepts the emergency task.

**Steps**

1. Rescuer receives emergency notification
2. Rescuer opens Emergency Request
3. Rescuer reviews case details:

* Location
* Snake identification (if available)
* Symptoms (if available)

1. Rescuer taps **“**Nhận nhiệm vụ**”** button
2. System updates status to Rescuer Assigned
3. System navigates to navigation mapScreen

**System Actions**

* System updates Member
* System updates Operator
* System tracks rescuer location

**Expected Result**

* Rescuer accepts mission
* Member sees rescuer info
* ETA displayed

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.19 Mission Detail - SOS Screen

**Step 5: Rescuer Travels to Emergency Location**

Rescuer travels to the Member's location.

**Steps**

1. System opens navigation map
2. Rescuer taps “Bắt đầu di chuyển” button
3. Rescuer travels to Member location
4. Rescuer updates status to On The Way

**System Actions**

* System tracks rescuer location
* Member sees rescuer movement
* ETA updated automatically

**Expected Result**

* Rescuer en route
* Member can track rescuer

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.20 Navigation Map Screen

**Step 6: Rescuer Provides First Aid**

Rescuer arrives at the location and provides first aid support.

**Steps**

1. Rescuer taps “Đã đến nơi” button
2. System opens on-scene support
3. Rescuer performs first aid

**Expected Result**

* Emergency handled
* Case ready for payment

![](data:image/png;base64...)

###### Figure 6.3.2.21 On-Scene support Screen

**Step 6.1: Find Nearby Hospital (Optional)**

After providing first aid, the rescuer can find nearby hospitals to transfer the member if necessary.

This step is optional and depends on the emergency severity.

![](data:image/png;base64...) ![](data:image/png;base64...)

###### Figure 6.3.2.22 Find Nearby Hospital

**Steps**

1. Rescuer taps “Tìm bệnh viện” button
2. System displays list of nearby hospitals
3. Rescuer selects suitable hospital
4. Rescuer taps “Chọn bệnh viện”
5. System opens navigation to selected hospital

**System Actions**

* System detects current location
* System retrieves nearby hospitals
* System calculates distance and travel time
* System opens navigation map

**Expected Result**

* Hospital list displayed
* Rescuer navigates to selected hospital
* Member receives continued support

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.23 Navigation to Hospital Screen

**Step 7: Complete Support Mission**

After providing first aid and optional hospital navigation, the rescuer completes the support mission.

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.24 Complete Support Mission

**Steps**

1. Rescuer taps "Hoàn thành hỗ trợ" button
2. System navigates to Mission Completion Screen
3. Rescuer provides support result
4. Rescuer uploads evidence images
5. Rescuer confirms completion
6. Rescuer taps “Xác nhận hoàn thành”
7. System navigates to Mission Success - Emergency Screen

**Support Information**

**Rescuer provides the following information:**

* Support result description
* Member condition after support
* Additional notes (optional)
* Evidence images

**System Actions**

* System stores support result
* System uploads evidence images
* System updates mission status to Completed
* System notifies Operator and Member

**Expected Result**

* Mission completed successfully
* Support result recorded
* Case ready for payment

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.25 Mission Completion Screen

![](data:image/png;base64...)

###### Figure 6.3.2.26 Mission Success - Emergency Screen

**Step 8: Payment Request**

After mission completion, system sends payment request to Member.

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.2.27 Member Incident Finished Screen

**Steps**

1. System generates service fee
2. Member receives payment notification
3. Member opens payment screen
4. Member taps “Thanh toán ngay” button
5. Member selects payment method
6. Member confirms payment

**Expected Result**

* Payment completed
* Case ready to close

![](data:image/png;base64...)

###### Figure 6.3.2.28 Member Payment Selection

![](data:image/png;base64...)

###### Figure 6.3.2.29 Payment Success

**Step 9: Case Completion**

After successful payment, the system closes the emergency case.

**System Actions**

* System updates case status to Completed
* System stores emergency history
* System updates payment settlement

**Expected Result**

* Emergency case completed
* Payment settled
* Case stored in history

![](data:image/png;base64...)

###### Figure 6.3.2.30 Activity History Screen (Member - tab “Sự cố”)

###

###

###

###

### 3.3 Snake Catching

**Purpose**

This workflow allows Members to request catching snakes and receive immediate support from rescuers.

**Step-by-Step Guide**

**Step 1: Member Creates Snake Catching Request**

This step allows Member to create a snake catching request when a snake is found and requires professional assistance.

**Step 1.1 Trigger Snake Catching Service**

Member starts the snake catching request from Homepage.

**Steps**

1. Member opens SnakeAid application
2. Member taps **"Cần bắt rắn"** button
3. System navigates to **Service Selection Screen**

**Expected Result**

* Service selection screen is displayed
* Member can select snake catching service

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.1 Member Homepage — "Cần bắt rắn" Button

![](data:image/png;base64...)

###### Figure 6.3.3.2 Snake Quantity Service Selection Screen

**Step 1.2 Select Snake Catching Type**

Member selects the snake catching type depending on the situation.

**Available Options**

* Single snake
* 2–5 snakes
* Snake nest

**Steps**

1. Member selects snake catching type
2. System updates service request type

**Expected Result**

* Snake catching type selected successfully and navigate correctly selected screen.

![](data:image/png;base64...)

###### Figure 6.3.3.3 Snake Report Detail Screen

**Step 1.3 Provide Snake Information**

Member can provide snake information using one of the following flows.

These actions are optional and do not require sequential order.

**Option 1: Snake Detection by Image (AI Detection)**

Member provides snake image for AI detection.

**Steps**

1. Member taps “Ảnh rắn”
2. Camera screen is displayed
3. Member captures snake image
4. System processes AI detection
5. System displays detected snake species

**Expected Result**

* Snake species detected successfully
* Snake information attached to request

![](data:image/png;base64...)

###### Figure 6.3.3.4 Snake Report Detail Screen (Upload Image for AI Snake Detection)

![](data:image/png;base64...)

###### Figure 6.3.3.5 Snake Report Detail Screen (Snake AI Detection Result)

**Option 2: Snake Selection by Location**

Member selects snake species based on nearby location.

Steps

1. Member taps "Chọn Loài Rắn" Tab
2. System displays nearby snake species
3. Member selects snake species

Expected Result

* Snake species selected successfully

![](data:image/png;base64...)

###### Figure 6.3.3.6 Snake Report Detail Screen (Snake Selection by Location)

**Step 1.4 Fill Request Information**

Member provides additional request details.

**Fields**

* GPS Location
* Address detail (Specific location)
* Provide image
* Note (Optional)

**Steps**

1. System automatically detects GPS location
2. Member reviews location on map
3. Member enters address detail
4. Member enters additional note (optional)

**Expected Result**

* Request information completed successfully

![](data:image/png;base64...)

###### Figure 6.3.3.7 Snake Report Detail Screen (Fill Full Request Form)

######

**Step 1.5 Submit Snake Catching Request**

Member submits snake catching request.

**Steps**

1. Member taps “Gửi Báo Cáo”
2. System validates request information
3. System creates snake catching request
4. System calculates estimated distance
5. System calculates estimated travel fee
6. System sets request status to Pending
7. System notifies Operator

**Expected Result**

* Snake catching request created successfully
* Request status set to Pending
* Operator receives notification

![](data:image/png;base64...)

###### Figure 6.3.3.8 Snake Catching Success Screen

**Step 2: Member Pays Travel Fee (Round 1 Payment)**

Member pays travel fee before rescuer assignment.

**Steps**

1. Member taps “Thanh Toán Phí Di Chuyển”
2. System navigates to Activity Detail Screen
3. Member selects payment method
4. Member confirms payment
5. System processes payment

**Expected Result**

* Payment successful
* Payment recorded

![](data:image/png;base64...)

###### Figure 6.3.3.9 Member Payment Selection

![](data:image/png;base64...)

###### Figure 6.3.3.10 Payment Success For Travel Fee

**Step 2.1: Member Cancels Request (Optional)**

Member can cancel the snake catching request after creating the request and before rescuer assignment.

After submitting the snake catching request, the Member may cancel the request if the situation changes or the snake is no longer present.

**Conditions**

* Request status is Pending or Confirmed
* No rescuer assigned yet

**Steps**

1. Member opens Snake Catching Request Detail Screen
2. Member taps “Hủy đơn”
3. System displays confirmation dialog
4. Member confirms cancellation
5. System updates request status to Cancelled

**Expected Result**

* Request cancelled successfully
* Operator receives cancellation notification
* Request removed from active list

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.11 Member Cancel Request Button

![](data:image/png;base64...)

###### Figure 6.3.3.12 Cancel Request Confirmation Dialog (Member)

**Step 3: Operator Confirms Catching Case**

After a catching request is created, the system notifies the Operator to confirm the catching request case.

The Operator reviews the catching information and contacts the Member to confirm the situation.

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.13 Operator Dashboard (“Bắt Rắn” Tab)

**System Actions**

* System creates Catching Case
* Status = Pending
* System notifies Operator
* Operator receives Catching request

**Operator Actions**

1. Login role Operator on web
2. Operator opens “Bắt rắn” Tab
3. Operator reviews catching request information
4. Operator checks:

* Member location
* Snake identification (if available)

1. Operator contacts Member (if necessary)
2. Operator verifies catching request case
3. Operator click “Xác nhận yêu cầu” to updates status “Confirmed”

**Expected Result**

* Catching case confirmed
* Case ready for rescuer assignment

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.14 Catching Case Detail — Pending Status (Operator View)

**Step 4: Operator Assigns Rescuer**

After confirming the case, the Operator assigns a Rescuer to handle the catching.

![](data:image/png;base64...)

###### Figure 6.3.3.15 Catching Case Detail — Confirmed Status (Operator View)

**Steps**

1. Operator selects **“**Điều phối đội cứu hộ**”** button
2. System displays available rescuers nearby
3. Operator selects suitable rescuer
4. Operator confirms assignment
5. System updates case status to “Assigned”

**System Actions**

* System sends notification to Rescuer
* System sends ETA to Member
* System updates catching status

**Expected Result**

* Rescuer assigned
* Member receives rescuer information
* Rescuer receives catching request

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.16 Assign Rescuer Screen

**![](data:image/png;base64...)**

###### Figure 6.3.3.17 Catching Case Detail — Assigned Status (Operator View)

**Step 5: Rescuer Receives Assignment**

After the Operator assigns the rescuer, the system automatically assigns the mission to the rescuer without requiring acceptance.

**Steps**

1. Operator assigns rescuer
2. System sends notification to Rescuer
3. System updates status to Preparing
4. Rescuer receives assignment notification
5. Rescuer taps "Xem chi tiết"
6. System navigates to Mission Detail Screen

**Expected Result**

* Rescuer receives mission assignment
* Status updated to Preparing
* Rescuer can start travel (if member paid travel fee)

![](data:image/png;base64...)

###### Figure 6.3.3.18 Request Catching Modal

![](data:image/png;base64...)

###### Figure 6.3.3.19 Accept Request Screen

**Step 5.1: Rescuer Cancels Assignment (Optional)**

Rescuer can cancel the assigned snake catching request before starting travel.

**Description**

After receiving assignment, the rescuer may cancel the request if they are unavailable, too far away, or unable to handle the situation.

**Conditions**

* Request status is Assigned
* Rescuer has not started travel yet

**Steps**

1. Rescuer receives assignment notification
2. Rescuer opens **Accept Request Screen**
3. Rescuer taps “Hủy đơn”
4. System displays confirmation dialog
5. Rescuer confirms cancellation
6. System updates rescuer assignment status to **Cancelled**
7. System notifies Operator

**Expected Result**

* Assignment cancelled successfully
* Operator receives cancellation notification
* Request returns to Confirmed status
* Operator assigns another rescuer

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.20 Rescuer Cancel Request Button

![](data:image/png;base64...)

###### Figure 6.3.3.21 Cancel Request Confirmation Dialog (Rescuer)

**Step 6: Rescuer Travels to Location**

**Steps**

1. Rescuer taps “Bắt đầu di chuyển”
2. Status updated to En Route
3. Rescuer travels to location
4. Rescuer taps “Đã Đến nơi”

**Expected Result**

• Status updated to **Arrived**

![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.3.22 En-Route Screen

**Step 7: Rescuer Captures Snake**

**Steps**

1. Rescuer performs snake catching
2. Rescuer captures evidence photo
3. Rescuer uploads evidence
4. Rescuer taps “Hoàn Thành Bắt Rắn”

**Expected Result**

* Status updated to **Finished**

![](data:image/png;base64...)

###### Figure 6.3.3.23 Tracking Screen

**Step 8: Rescuer Confirms Completion**

Rescuer confirms mission completion and provides actual catching information for cost calculation.

**Description**

After capturing the snake, the rescuer must confirm the actual result. The system will use this information to calculate the final service fee.

**Fields**

* Actual snake species captured
* Number of snakes captured
* Catching area (Indoor / Outdoor / Roof / Garden / Other)
* Additional notes (Optional)

**Steps**

1. System navigates to Result Confirmation Screen
2. Rescuer selects Actual Snake Species
3. Rescuer selects Number of Snakes Captured
4. Rescuer selects Catching Area
5. Rescuer enters additional notes (optional)
6. Rescuer uploads evidence images
7. Rescuer taps **“**Gửi kết quả cho khách hàng”
8. Rescuer taps “Gửi ngay”
9. System navigates to Mission Success - Snake Catching Screen

**Expected Result**

* Completion information submitted
* System calculates actual service fee
* Status updated to Finished
* Member receives final payment request

![](data:image/png;base64...)

###### Figure 6.3.3.24 Result Confirmation Screen

![](data:image/png;base64...)

###### Figure 6.3.3.25 Mission Success - Snake Catching Screen

**Step 9: Member Pays Service Fee**

**Steps**

1. Member opens Activity Screen Detail for snake catching
2. Member reviews service details
3. Member taps “Thanh toán dịch vụ”
4. Member selects payment method
5. Payment processed

**Expected Result**

* Payment completed
* Status updated to Completed

![](data:image/png;base64...)

###### Figure 6.3.3.26 Member Payment Selection

![](data:image/png;base64...)

###### Figure 6.3.3.27 Payment Catching Success

**Step 10: Case Completed**

**Steps**

1. System updates status to Completed
2. System displays rescuer rating
3. System sends completion notification
4. System stores request history

**Expected Result**

* Case completed successfully

![](data:image/png;base64...)

###### Figure 6.3.3.28 Activity History Screen (Member - tab “Bắt rắn”)

###

###

###

###

###

### 3.4 Scheduled Consultation

**Purpose**

This workflow allows Members to schedule consultation sessions with Experts at a selected date and time. The system supports booking, payment, consultation session, and completion.

**Step-by-Step Guide**

**Step 1: Browse Expert Directory**

Member opens the Expert directory to view available experts.

**Steps**

1. Member opens Expert Directory
2. System displays expert list
3. Member searches or filters experts
4. Member selects an Expert
5. Member taps “Đặt Tư Vấn” button

**System Actions**

* System loads expert profiles
* System displays availability information
* System shows consultation services

**Expected Result**

* Member selects expert
* Member navigates to Expert Detail Screen

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.4.1 Expert Directory Screen

![](data:image/png;base64...)

###### Figure 6.3.4.2 Expert Detail Screen

**Step 2: Select Consultation Service**

Member selects consultation service and chooses schedule booking.

**Steps**

1. Member taps "Chọn Đặt Lịch"
2. System navigates to Schedule Selection Screen
3. System displays available dates
4. Member selects preferred date
5. System displays available time slots
6. Member selects time slot

**System Actions**

* System retrieves expert availability
* System validates slot availability
* System locks selected slot temporarily

**Expected Result**

* Time slot selected
* Member proceeds to booking details

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.4.3 Schedule Selection Screen

**Step 3: Enter Consultation Details**

Member enters consultation details before confirming booking.

**Steps**

1. Member enters consultation details
2. Member taps "Xác Nhận Đặt Lịch"

**System Actions**

* System validates booking information
* System creates scheduled booking
* Booking status = Pending Payment
* System locks selected slot

**Expected Result**

* Booking created successfully
* Member navigates to payment screen

![](data:image/png;base64...)

###### Figure 6.3.4.4 Consultation Detail Form Screen

**Step 4: Confirm & Payment**

Member confirms payment for scheduled consultation.

**Steps**

1. Member reviews consultation summary
2. Member taps "Xác Nhận & Thanh Toán"
3. Member selects payment method
4. Member confirms payment

**System Actions**

* System generates consultation fee
* Payment gateway processes transaction
* System holds payment in escrow
* Booking status updated to Confirmed

**Expected Result**

* Payment successful
* Consultation scheduled successfully

![](data:image/png;base64...)

###### Figure 6.3.4.5 Confirm & Payment Screen

**Step 5: Consultation Highlight**

After payment, Member is redirected to consultation home.

**Steps**

1. System redirects Member to Consultation Home
2. System highlights upcoming consultation
3. Member views consultation details

**System Actions**

* System stores booking information
* System displays upcoming consultation

**Expected Result**

* Member sees scheduled consultation
* Member waits for consultation time

*![](data:image/png;base64...)*

###### Figure 6.3.4.6 Consultation Home Screen

**Step 6: Join Waiting Room**

At consultation time, Member and Expert join waiting room.

**Steps**

1. Member taps Join Consultation
2. System navigates to Waiting Room
3. Expert joins waiting room
4. System enables video session

**System Actions**

* System verifies consultation time
* System connects video session
* System notifies participants

**Expected Result**

* Member and Expert ready for consultation

![](data:image/png;base64...)

###### Figure 6.3.4.7 Waiting Room Screen (Member)

**![](data:image/png;base64...)**

###### Figure 6.3.4.8 Waiting Room Screen (Expert)

**Step 7: Video Consultation Session**

Member and Expert conduct consultation session.

**Steps**

1. System starts video session
2. Member taps “Vào Phòng” to enter the room
3. Expert taps “Bắt Đầu Tư Vấn” to enter the room
4. Expert provides consultation advice
5. Expert or Member ends session

**System Actions**

* System records consultation duration
* System stores consultation session data

**Expected Result**

* Consultation completed

![](data:image/png;base64...)

###### Figure 6.3.4.9 Video Consultation Screen

**Step 8: Consultation Completion**

After session ends, system completes consultation.

**System Actions**

* System marks consultation as Completed
* System releases escrow payment
* System stores consultation history

**Expected Result**

* Consultation completed
* Payment settled

**Step 9: Rating and Feedback**

Member rates Expert after consultation.

**Steps**

1. System displays rating screen
2. Member selects rating
3. Member enters feedback
4. Member submits rating

**System Actions**

* System stores rating
* System updates expert rating score

**Expected Result**

* Rating submitted successfully

![](data:image/png;base64...)

###### Figure 6.3.4.10 Rating Screen

### 3.5 Instant Consultation

**Purpose**

This workflow supports real-time consultation with available Experts. Members can immediately connect with an online Expert for urgent consultation without scheduling in advance.

**Step-by-Step Guide**

**Step 1: Select Instant Consultation**

Member selects an available Expert for instant consultation.

**Steps**

1. Member opens Expert Directory
2. Member selects an available Expert
3. Member taps "Chọn Tư Vấn Ngay"
4. System validates Expert availability

**System Actions**

* System checks Expert online status
* System checks Expert availability
* System validates consultation readiness

**Expected Result**

* Expert availability confirmed
* Member proceeds to consultation details

![](data:image/png;base64...)![](data:image/png;base64...)![](data:image/png;base64...)

###### Figure 6.3.5.1 Expert Detail Screen — Instant Consultation

**Step 1.1: Expert Unavailable (Alternative Flow)**

If the selected Expert is unavailable, the system disables the “Chọn Tư Vấn Ngay” option.

**Trigger Condition**

* Expert is offline

**System Actions**

* System disables the “Chọn Tư Vấn Ngay” option

**Member Actions**

1. Member selects another Expert
    OR
2. Member switches to Scheduled Consultation

**Expected Result**

* Member selects alternative option

![](data:image/png;base64...)

###### Figure 6.3.5.2 Immediate Consultation Disabled

**Step 2: Enter Consultation Details**

Member enters consultation information before confirming payment.

**Steps**

1. Member enters consultation details
2. Member taps "Xác Nhận & Thanh Toán"

**System Actions**

* System validates consultation details
* System prepares emergency consultation request

**Expected Result**

* Consultation request ready for payment

![](data:image/png;base64...)

###### Figure 6.3.5.3 Consultation Detail Form Screen

**Step 3: Payment Processing**

Member completes payment before starting consultation.

**Steps**

1. Member selects payment method
2. Member confirms payment
3. Payment gateway processes transaction

**System Actions**

* Payment gateway processes payment (Wallet / PayOS)
* System holds escrow payment
* System returns payment status

**Expected Result**

* Payment successful
* Instant consultation request created

![](data:image/png;base64...)

###### Figure 6.3.5.4 Confirm & Payment Screen

**Step 4: Pending Expert Response**

After payment, system waits for Expert response.

**Steps**

1. System creates consultation request
2. Status = PendingExpertResponse
3. System notifies Expert
4. Member waits for Expert response

**System Actions**

* System sends realtime notification
* System updates consultation status
* System starts response timer

**Expected Result**

* Expert receives consultation request
* Member sees waiting status

![](data:image/png;base64...)

###### Figure 6.3.5.5 Waiting Expert Response Screen

**Step 5: Expert Accepts / Rejects Request**

Expert reviews consultation request.

**Steps**

1. Expert receives consultation request
2. Expert reviews consultation details
3. Expert selects:

* Accept
* Reject

**Case 1: Expert Accepts**

* System connects Member and Expert
* System navigates to waiting room

**Case 2: Expert Rejects**

* System notifies Member
* System allows Member to select another Expert

**System Actions**

* System updates consultation status
* System sends realtime update

**Expected Result**

* Expert accepts or rejects consultation

**![](data:image/png;base64...)**

###### Figure 6.3.5.6 Expert Global Emergency Popup Listener

**Step 6: Join Waiting Room**

Member and Expert enter waiting room.

**Steps**

1. System navigates both parties to waiting room
2. System prepares video session
3. Member taps “Vào Phòng” to join waiting room
4. Expert taps “Bắt Đầu Tư Vấn” to join waiting room

**System Actions**

* System initializes video session
* System verifies connection

**Expected Result**

* Member and Expert ready for consultation

![](data:image/png;base64...)**![](data:image/png;base64...)**

###### Figure 6.3.5.7 Member and Expert Waiting Room Screen

**Step 7: Instant Video Consultation**

Member and Expert conduct real-time consultation.

**Steps**

1. Video session starts
2. Member explains situation
3. Expert provides consultation advice
4. Expert or Member ends session

**System Actions**

* System records session duration
* System stores consultation information

**Expected Result**

* Consultation completed successfully

![](data:image/png;base64...)

###### Figure 6.3.5.8 Video Consultation Screen

**Step 8: Consultation Completion**

After session ends, system completes consultation.

**System Actions**

* System marks consultation as Completed
* System settles escrow payment
* System stores consultation history

**Expected Result**

* Consultation completed
* Payment settled

**Step 9: Rating and Feedback**

Member submits rating after consultation.

**Steps**

1. System displays rating screen
2. Member selects rating score
3. Member enters feedback
4. Member submits rating

**System Actions**

* System stores rating
* System updates expert rating

**Expected Result**

* Rating submitted successfully

![](data:image/png;base64...)

###### Figure 6.3.5.9 Rating Screen

###

###

###

###

###

###

### 3.6 System Administration

**Purpose**

This workflow describes administrative management features that allow administrators to monitor system operations, manage users, configure domain data, and control system activities.

Administrators use the Admin Dashboard to supervise the entire SnakeAid system.

**Step-by-Step Guide**

**Step 1: Access Admin Dashboard**

Admin accesses the Admin Dashboard after successful sign-in.

**Steps**

1. Admin logs into Admin Web Portal
2. System authenticates Admin account
3. System navigates to Admin Dashboard or Click to “Bảng điều khiển”

**System Actions**

* System validates admin credentials
* System loads dashboard data
* System displays system analytics

**Expected Result**

* Admin successfully accesses dashboard
* Admin views system overview

![](data:image/png;base64...)

###### Figure 6.3.5.1 Admin Dashboard

**Step 2: User management**

Admin manages users in the system.

**Dashboard Information**

* Active users
* Inactive users
* New user per month

**Steps:**

1. Admin logs into Admin Web Portal
2. Admin click “Quản lý người dùng” on menu
3. System navigates to User Management screen
4. Click an account to see user details

**System Actions**

* System loads analytics widgets
* System displays real-time data

**Expected Result**

* Admin understands system status

*![](data:image/png;base64...)*

###### Figure 6.3.5.2 User Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.3 User Detail Screen

**Step 2.1: Lock User Account**

Admin can lock user accounts when detecting violations or suspicious activities.

**Steps**

1. Admin opens User Detail Screen
2. Admin clicks "Khóa tài khoản" button
3. System displays Lock Account Dialog
4. Admin enters lock reason
5. Admin clicks "Xác nhận khóa"
6. System updates account status to Blocked

**System Actions**

* System stores lock reason
* System updates user status to Blocked
* System disables user access
* System records admin action log

**Expected Result**

* User account locked successfully
* User cannot access system
* Lock reason stored for audit

*![](data:image/png;base64...)*

###### Figure 6.3.5.4 Lock User Account Dialog

**Step 4: Manage Rescuer Scheduling**

Admin manages rescuer schedules and shift assignments.

**Steps**

1. Admin click to “Quản lý lịch làm việc” on menu
2. System navigates to Workshifts Management Screen
3. Admin clicks "Chi tiết" for the slot to assign
4. Admin tick rescuers to join the slot
5. Admin clicks "Gán 1 cứu hộ viên" or "Gán hàng loạt" to assign rescuers to shifts
6. System updates the rescuer assigned to the slot

**System Actions**

* System updates staff schedule
* System notifies assigned rescuers

**Expected Result**

* Staff schedules updated

*![](data:image/png;base64...)*

###### Figure 6.3.5.5 Workshifts Management Screen

Admin can manage shift setup, then open one cell "Chi tiết" to assign or update rescuers.

![](data:image/png;base64...)

###### Figure 6.3.5.6 Shift Assignment Details Modal

Admin can open "Quản lý sự cố" and "Quản lý yêu cầu bắt rắn" to inspect requests and progress by status.

**Step 5: Manage Emergency Requests**

Admin manages system requests.

**Available Management Modules**

* Manage snake bite (“Quản lý sự cố”)
* Manage snake catching (“Quản lý bắt rắn”)
* Manage consultation (“Quản lý tư vấn”)

**Steps**

1. Admin navigates to request management module
2. Admin filters requests
3. Admin reviews request details
4. Admin monitors request progress

**Expected Result**

* Requests monitored successfully

*![](data:image/png;base64...)*

###### Figure 6.3.5.7 Incidents Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.8 Incident Detail Modal

*![](data:image/png;base64...)*

###### Figure 6.3.5.9 Snake Catching Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.10 Snake Catching Detail Modal

*![](data:image/png;base64...)*

###### Figure 6.3.5.11 Consultations Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.12 Consultation Detail Modal

**Step 6: Manage Domain Data**

Admin manages system domain data.

**Available Modules**

* Snake Management (“Quản lý loài rắn”)
* Antivenoms Management (“Quản lý huyết thanh”)
* Treatment Facilities Management (“Quản lý cơ sở điều trị”)

**Steps**

1. Admin navigates to domain management
2. Admin creates new data
3. Admin updates existing data
4. Admin deletes data

**System Actions**

* System updates domain data
* System refreshes system information

**Expected Result**

* Domain data updated

*![](data:image/png;base64...)*

###### Figure 6.3.5.13 Snakes Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.14 Antivenoms Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.15 Treatment Facilities Management Screen

**Step 7: Financial Management**

Admin controls financial operations.

**Available Modules**

* Quản lý giao dịch
* Duyệt rút tiền

**Steps**

1. Admin opens “Quản Lý Giao Dịch”
2. Admin reviews transaction details
3. Admin opens “Duyệt rút tiền”
4. Admin approves withdrawal request
5. Admin confirms action

**System Actions**

* System updates transaction status
* System processes withdrawal

**Expected Result**

* Financial operations managed successfully

![](data:image/png;base64...)

###### Figure 6.3.5.16 Transactions & Withdrawals Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.17 Withdrawals Management Screen - “Đã Xử Lý” Tab

*![](data:image/png;base64...)*

###### Figure 6.3.5.18 Withdrawals Management Screen - “Đang Xử Lý” Tab

**Step 8: Content Management**

Admin manages learning and content modules.

**Available Modules**

* Lesson Management (“Quản lý bài học”)
* Blogs Management (“Quản lý bài viết”)

**Steps**

1. Admin navigates to content management
2. Admin creates new content
3. Admin edits content
4. Admin deletes content
5. Admin moderates content

**Expected Result**

* Content updated successfully

*![](data:image/png;base64...)*

###### Figure 6.3.5.19 Lessons Management Screen

*![](data:image/png;base64...)*

###### Figure 6.3.5.20 Blogs Management Screen

**Step 9: AI Recognition Monitoring**

Admin reviews AI snake recognition records.

**Steps**

1. Admin navigates to “Quản lý ảnh báo cáo rắn”
2. Admin filters recognition records
3. Admin reviews recognition details

**System Actions**

* System retrieves AI recognition logs

**Expected Result**

* AI recognition monitored

*![](data:image/png;base64...)*

###### Figure 6.3.5.21 Report Media Management Screen

**Step 10: Dynamic Configuration Management**

Admin manages runtime configuration.

**Steps**

1. Admin navigates to “Quản lý cấu hình động”
2. Admin updates configuration values
3. Admin applies reload action

**System Actions**

* System reloads configuration
* System applies changes

**Expected Result**

* Configuration updated successfully

![](data:image/png;base64...)

###### Figure 6.3.5.22 Settings Management Screen