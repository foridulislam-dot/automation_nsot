আগের চ্যাপ্টারে আমরা নেটওয়ার্ক ইনভেন্টরি তৈরি করলাম - সাইট, ডিভাইস, ইন্টারফেস, আইপি। এখন সময় এসেছে এই সব কিছুকে একসাথে জোড়া লাগানোর। কোন ডিভাইস কোন ডিভাইসের সাথে কানেক্টেড, কোন ক্যাবল কোথায় গেছে, আপস্ট্রিম প্রোভাইডার কে - এসব ডকুমেন্ট করা। আর একইসাথে দেখব কীভাবে Nautobot-কে আপনার নিজের মতো কাস্টমাইজ করবেন।

এই চ্যাপ্টারটা একটু আলাদা। এখানে আমরা বেশ কিছু টপিক কভার করব যেগুলো আপনার Nautobot সেটআপকে কমপ্লিট করে দেবে। শুরু করি প্রোভাইডার আর সার্কিট দিয়ে।

### Provider এবং Circuit Management

আপনার আইএসপি চালাতে হলে ইন্টারনেট লাগবে। আর সেই ইন্টারনেট আসে আপস্ট্রিম প্রোভাইডার থেকে। বাংলাদেশে হতে পারে BTCL, Summit Communications, Fiber@Home, বা অন্য কোনো IIG/ICSP লেয়ার থেকে। এই প্রোভাইডারদের ডিটেইলস আর তাদের কাছ থেকে নেওয়া লিংক (Circuit) Nautobot-এ রাখা খুবই গুরুত্বপূর্ণ।

### Provider তৈরি করা

চলুন QuickNet Bangladesh-এর জন্য তাদের প্রাইমারি প্রোভাইডার এড করি। ধরুন তারা BTCL থেকে ইন্টারনেট নিয়েছে।

নেভিগেশন বারে **Circuits → Providers** যান। **+ Add** বাটনে ক্লিক করুন।

একটা ফর্ম আসবে। পূরণ করুন:

```
Name: BTCL
ASN: 1x494
Account Number: QN-BTCL-2024-001
Portal URL: https://isp.btcl.gov.bd
NOC Contact: +880 2-955xx55
Admin Contact: noc@btcl.gov.bd
Comments: Tartiary upstream provider. Contract started Jan 2024.
```

**ASN** হলো Autonomous System Number। এটা প্রতিটা ইন্টারনেট সার্ভিস প্রোভাইডারের একটা ইউনিক নাম্বার। BTCL-এর ASN হলো 1x494। আপনি গুগল করলেই যেকোনো প্রোভাইডারের ASN পেয়ে যাবেন।

**Account Number** হলো আপনার প্রোভাইডারের কাছে আপনার অ্যাকাউন্ট নাম্বার। এটা বিলিং বা সাপোর্ট টিকেটের সময় কাজে লাগে।

**Create** বাটনে ক্লিক করুন। আপনার প্রথম প্রোভাইডার এড হয়ে গেল।

একইভাবে যদি ব্যাকআপ প্রোভাইডার থাকে, সেটাও এড করে ফেলুন। যেমন:

```
Name: Summit Communications
ASN: 4x928
Account Number: QN-SUMMIT-2024-002
Portal URL: https://portal.summitcommunications.net
NOC Contact: +880 2-883x464
Admin Contact: noc@summitcommunications.net
Comments: Primary/backup provider
```

### Circuit Type তৈরি

Circuit এড করার আগে Circuit Type তৈরি করতে হবে। Circuit Type মানে হলো লিংকের ধরন। যেমন ফাইবার অপটিক, মাইক্রোওয়েভ, IPLC ইত্যাদি।

**Circuits → Circuit Types** যান। **+ Add** ক্লিক করুন।

```
Name: Fiber Optic
Description: Fiber optic connectivity
```

**Create** করুন। একইভাবে আরো কয়েকটা টাইপ বানান:

```
Name: Microwave Link
Description: Point-to-point microwave

Name: IPLC
Description: International Private Leased Circuit
```

### Circuit তৈরি করা

এখন আসল সার্কিট তৈরি করব। ধরুন QuickNet তাদের মিরপুর পপে BTCL থেকে একটা ১০ Gbps ফাইবার লিংক নিয়েছে।

**Circuits → Circuits** যান। **+ Add** ক্লিক করুন।

```
Circuit ID: BTCL-MIR-001
Provider: BTCL
Type: Fiber Optic
Status: Active
Install Date: 2024-01-15
Commit Rate: 10000 Mbps
Description: Primary 10G uplink from Mirpur POP
```

**Circuit ID** হলো একটা ইউনিক আইডেন্টিফায়ার। একটা স্ট্যান্ডার্ড ফরম্যাট মেনে চলুন। আমরা ব্যবহার করেছি [Provider]-[Location]-[Number] ফরম্যাট।

**Commit Rate** হলো গ্যারান্টিড ব্যান্ডউইথ। এটা Mbps-এ দিতে হবে। ১০ Gbps মানে ১০০০০ Mbps।

এখন **Create and Add Another** বাটনে ক্লিক করুন (যদি একাধিক সার্কিট এড করতে চান)। অথবা শুধু **Create** ক্লিক করুন।

সার্কিট তৈরি হওয়ার পরে আপনি ডিটেইল পেজে যাবেন। এখানে আরো কিছু ইনফরমেশন এড করতে পারবেন।

### Circuit Termination - সার্কিট কোথায় শেষ হচ্ছে

একটা সার্কিটের দুটো প্রান্ত থাকে। একদিকে প্রোভাইডার, অন্যদিকে আপনার ডিভাইস। এই দুই প্রান্তকে বলা হয় Termination।

Circuit ডিটেইল পেজে **Terminations** ট্যাবে যান। **+ Add Termination** ক্লিক করুন।

প্রথম টার্মিনেশন (প্রোভাইডারের দিক):

```
Term Side: A
Location: (খালি রাখুন - প্রোভাইডার সাইড)
Port Speed: 10 Gbps
Upstream Speed: 10000 Mbps
Cross Connect ID: BTCL-XC-12345 (যদি প্রোভাইডার দিয়ে থাকে)
PP Info: PP-BTCL-MIR-001 (Patch Panel ইনফো)
Description: BTCL side termination
```

**Create** করুন।

এখন দ্বিতীয় টার্মিনেশন (আপনার ডিভাইসের দিক):

**+ Add Termination** আবার ক্লিক করুন।

```
Term Side: Z
Location: Mirpur POP
Device: R-MIR-CORE-01
Interface: sfpplus1
Port Speed: 10 Gbps
Description: Connected to core router uplink port
```

**Create** করুন। এখন আপনার সার্কিট সম্পূর্ণভাবে ডকুমেন্ট করা। যে কেউ দেখলেই বুঝবে এই সার্কিট কোথা থেকে এসে কোথায় কানেক্টেড।

একটা প্র্যাক্টিক্যাল টিপ: Comments ফিল্ডে কন্ট্রাক্ট ডিটেইলস লিখে রাখুন। যেমন:

```
Comments:
- Contract Start: 15-Jan-2024
- Contract Duration: 3 years
- Monthly Cost: BDT 500,000
- Contract Renewal Date: 15-Jan-2027
- Sales Contact: Mr. Karim, +880 17xx-123456
- Support Ticket Email: support@btcl.gov.bd
```

এই ইনফরমেশন পরে অনেক কাজে লাগবে। বিশেষ করে যখন কন্ট্রাক্ট রিনিউয়ালের সময় আসবে।

## Cable Management - তার কোথায় গেল?

নেটওয়ার্কে সবচেয়ে বিরক্তিকর জিনিস হলো ক্যাবল ম্যানেজমেন্ট। কোন তার কোথায় গেছে, কোন পোর্ট কোন পোর্টে কানেক্টেড - এগুলো ট্র্যাক করা। Nautobot-এ এটা খুবই সহজ।

### Cable তৈরি করা

ধরুন আপনার কোর রাউটার (R-MIR-CORE-01) থেকে একটা ডিস্ট্রিবিউশন সুইচে (SW-MIR-DIST-01) একটা ফাইবার ক্যাবল আছে।

দুটো উপায়ে ক্যাবল তৈরি করতে পারেন।

**পদ্ধতি ১: Device Interface থেকে**

R-MIR-CORE-01 ডিভাইসে যান। **Interfaces** ট্যাবে ক্লিক করুন। ধরুন sfpplus2 পোর্ট দিয়ে সুইচে যাচ্ছে। এই ইন্টারফেসে ক্লিক করুন।

ইন্টারফেস ডিটেইল পেজে **Connect Cable** বাটন দেখবেন। ক্লিক করুন।

একটা ফর্ম আসবে:

```
Termination A (Already filled):
Device: R-MIR-CORE-01
Interface: sfpplus2

Termination Z:
Device: SW-MIR-DIST-01
Interface: sfpplus1

Cable Type: SMF (Single-Mode Fiber)
Status: Connected
Color: Blue
Length: 5 (meters)
Label: CORE-TO-DIST-01
```

**Create** ক্লিক করুন। ক্যাবল তৈরি হয়ে গেল।

এখন যদি কোনো একটা ডিভাইসের ইন্টারফেসে যান, দেখবেন "Cable Connected" দেখাচ্ছে। ক্লিক করলে পুরো ক্যাবল ডিটেইলস দেখাবে।

**পদ্ধতি ২: সরাসরি Cable তৈরি**

**Devices → Cables** যান। **+ Add** ক্লিক করুন।

```
Termination A Type: Interface
Termination A - Device: R-MIR-CORE-01
Termination A - Interface: ether3

Termination Z Type: Interface
Termination Z - Device: SW-MIR-ACC-01
Termination Z - Interface: ether1

Type: CAT6
Status: Connected
Color: Yellow
Length: 15
Label: CORE-TO-ACC-01
```

এভাবে প্রতিটা কানেকশন ডকুমেন্ট করুন।

### Cable Color Coding

একটা ভালো প্র্যাক্টিস হলো ক্যাবলে কালার কোডিং করা। যেমন:

- **Blue**: Fiber (Single-Mode)
- **Orange**: Fiber (Multi-Mode)  
- **Yellow**: Cat6 Ethernet
- **Red**: Uplink/Critical
- **Green**: Access/Customer
- **Gray**: Management

আপনার নিজের স্ট্যান্ডার্ড বানান আর সেটা মেনে চলুন।

### Trace Cable - ক্যাবল ফলো করা

Nautobot-এ একটা দারুণ ফিচার আছে - Cable Trace। ধরুন একটা কাস্টমার রিপোর্ট করল তার নেট নেই। আপনি জানতে চান এই কাস্টমারের কানেকশন কোন কোন ডিভাইস দিয়ে যাচ্ছে।

যেকোনো ইন্টারফেসে গিয়ে **Trace** বাটনে ক্লিক করুন। Nautobot পুরো পাথ দেখাবে - এই পোর্ট থেকে শুরু করে শেষ পর্যন্ত কোথায় গেছে, মাঝে কোন কোন ডিভাইস আছে।

এটা ট্রাবলশুটিং করার সময় অমূল্য।

## Custom Fields - নিজের মতো করে সাজান

Nautobot-এ অনেক বিল্ট-ইন ফিল্ড আছে। কিন্তু কখনো কখনো আপনার নিজস্ব কিছু ইনফরমেশন স্টোর করতে হবে যেটা ডিফল্টে নেই। এজন্য আছে Custom Fields।

### কখন Custom Field লাগে?

কিছু উদাহরণ:

১. **Warranty Expiry Date:** প্রতিটা ডিভাইসের ওয়ারেন্টি কবে শেষ হবে সেটা ট্র্যাক করতে চান।

২. **Purchase Order Number:** ডিভাইস কেনার সময় PO নাম্বার রেকর্ড করতে চান।

৩. **Power Consumption:** প্রতিটা ডিভাইস কত ওয়াট খায় সেটা জানতে চান ক্যাপাসিটি প্ল্যানিং এর জন্য।

৪. **Building Name:** কাস্টমার কোন বিল্ডিংয়ে আছে সেটা ট্র্যাক করতে চান।

চলুন একটা Custom Field তৈরি করি।

### Custom Field তৈরির ধাপ

ধরুন আমরা সব ডিভাইসে Warranty Expiry Date ফিল্ড যোগ করতে চাই।

**Extensibility → Custom Fields** যান। **+ Add** ক্লিক করুন।

```
Content Types: dcim | device (এটা সিলেক্ট করুন)
Label: Warranty Expiry Date
Key: warranty_expiry (এটা অটো-জেনারেট হবে, চাইলে এডিট করতে পারেন)
Type: Date
Description: Device warranty expiration date
Required: No (চেকবক্স আনচেক রাখুন)
Default: (খালি)
Weight: 100
```

**Content Types** খুবই গুরুত্বপূর্ণ। এটা দিয়ে বলছেন কোন অবজেক্টে এই ফিল্ড দেখাবে। আমরা `dcim | device` সিলেক্ট করেছি মানে Device অবজেক্টে দেখাবে।

**Type** অনেকগুলো অপশন আছে:
- Text
- Integer
- Boolean (Yes/No)
- Date
- URL
- Select (ড্রপডাউন)
- Multi-select

আমরা Date সিলেক্ট করেছি কারণ তারিখ স্টোর করব।

**Create** ক্লিক করুন।

এখন যেকোনো Device-এ গিয়ে Edit করুন। দেখবেন নতুন একটা ফিল্ড এসেছে - "Warranty Expiry Date"। এখানে তারিখ সিলেক্ট করতে পারবেন।

### আরেকটা উদাহরণ: Power Consumption

```
Content Types: dcim | device
Label: Power Consumption (Watts)
Key: power_watts
Type: Integer
Description: Device power consumption in watts
Required: No
```

এখন প্রতিটা ডিভাইসে পাওয়ার কনজাম্পশন এন্ট্রি করতে পারবেন। পরে একটা স্ক্রিপ্ট লিখে পুরো র‍্যাকের টোটাল পাওয়ার ক্যালকুলেট করতে পারবেন।

### Select Type Custom Field

আরেকটা দরকারি টাইপ হলো Select। ধরুন আপনি ডিভাইসের Purchase Source ট্র্যাক করতে চান।

```
Content Types: dcim | device
Label: Purchase Source
Key: purchase_source
Type: Selection
Description: Where the device was purchased from
Required: No
Choices:
  - Local Vendor
  - International Import
  - Rental/Lease
  - Donation
```

**Choices** ফিল্ডে প্রতি লাইনে একটা অপশন লিখুন। এখন ডিভাইস এডিট করার সময় ড্রপডাউন থেকে সিলেক্ট করতে পারবেন।

## Tags - সহজে ক্যাটাগরাইজ করুন

Tags হলো একটা শক্তিশালী টুল যা দিয়ে আপনি যেকোনো অবজেক্টকে লেবেল করতে পারেন। তারপর সহজে ফিল্টার করতে পারেন।

### Tag তৈরি করা

**Extensibility → Tags** যান। **+ Add** ক্লিক করুন।

```
Name: Production
Color: 4caf50 (সবুজ রঙ)
Description: Production/live devices
```

**Create** করুন। একইভাবে আরো কিছু ট্যাগ বানান:

```
Name: Staging
Color: ff9800 (কমলা)
Description: Staging/testing devices

Name: End-of-Life
Color: f44336 (লাল)
Description: Devices approaching end of life

Name: Critical
Color: 000000 (কালো)
Description: Mission-critical infrastructure

Name: Backup
Color: 9e9e9e (ধূসর)
Description: Backup/standby devices
```

### Tag ব্যবহার করা

এখন যেকোনো ডিভাইসে গিয়ে Edit করুন। নিচে **Tags** ফিল্ড দেখবেন। সেখানে ক্লিক করলে সব ট্যাগের লিস্ট আসবে। যেগুলো প্রযোজ্য সেগুলো সিলেক্ট করুন।

যেমন আপনার কোর রাউটারে দিতে পারেন: `Production`, `Critical`

একটা পুরোনো সুইচে দিতে পারেন: `Production`, `End-of-Life`

ট্যাগের সবচেয়ে বড় সুবিধা হলো ফিল্টারিং। ডিভাইস লিস্ট পেজে গিয়ে Tags ফিল্টার দিয়ে শুধু Critical ডিভাইসগুলো দেখতে পারবেন। অথবা যেসব ডিভাইস End-of-Life ট্যাগ করা আছে সেগুলো দেখে রিপ্লেসমেন্ট প্ল্যান করতে পারবেন।

## Saved Filters - বারবার একই ফিল্টার

প্রায়ই একই ধরনের সার্চ করতে হয়। যেমন "মিরপুর পপের সব Active ডিভাইস" বা "সব Critical Tagged রাউটার"। প্রতিবার ফিল্টার সেট করা বিরক্তিকর। এজন্য আছে Saved Filters।

### Saved Filter তৈরি

যেকোনো লিস্ট পেজে যান (যেমন Devices)। উপরে ফিল্টার সেট করুন। যেমন:

- Location: Mirpur POP
- Status: Active
- Role: Core Router

ফিল্টার করার পরে URL দেখুন। অনেকগুলো প্যারামিটার দেখবেন। এখন উপরে **Save Filter** অপশন দেখবেন (ভার্সন অনুযায়ী লোকেশন ভিন্ন হতে পারে)।

ক্লিক করুন। একটা নাম দিন:

```
Name: Mirpur Active Core Routers
```

**Save** করুন। এখন পরেরবার যখন আসবেন, এই ফিল্টার এক ক্লিকে অ্যাপ্লাই করতে পারবেন।

Saved Filters আপনার ব্যক্তিগত। অন্য ইউজার দেখবে না। কিন্তু আপনি যখনই লগইন করবেন, আপনার সব Saved Filters পাবেন।

## User Management - টিম কে কী করতে পারবে

একজন মানুষ একা সব কাজ করতে পারে না। টিম লাগবে। কিন্তু সবাইকে সব পারমিশন দেওয়া উচিত না। একজন জুনিয়র টেকনিশিয়ান হয়তো শুধু দেখতে পারবে, এডিট করতে পারবে না। আরেকজন সিনিয়র সব করতে পারবে।

### নতুন User তৈরি

আপনি যেহেতু Superuser হিসেবে লগইন আছেন, আপনি নতুন ইউজার বানাতে পারবেন।

উপরের ডান কোণায় আপনার ইউজারনেমে ক্লিক করুন। ড্রপডাউনে **Admin** লিংক দেখবেন। ক্লিক করুন।

এটা আপনাকে Django Admin ইন্টারফেসে নিয়ে যাবে। এখানে **Users** সেকশনে যান। **Add User** ক্লিক করুন।

```
Username: jahid
Password: (একটা স্ট্রং পাসওয়ার্ড দিন)
```

**Save** করুন। তারপর আরো ডিটেইলস যোগ করুন:

```
First Name: Jahid
Last Name: Hassan
Email: jahid@quicknet.bd
Active: ✓ (চেক করুন)
Staff Status: ✓ (চেক করুন - এটা দিলে Nautobot UI অ্যাক্সেস পাবে)
Superuser Status: (আনচেক রাখুন - সবাইকে Superuser বানাবেন না)
```

**Save** করুন।

### Group তৈরি এবং Permission সেট

এখন Group তৈরি করুন। Django Admin-এ **Groups** সেকশনে যান। **Add Group** ক্লিক করুন।

```
Name: Network Operators
```

এখন Permissions সিলেক্ট করুন। অনেকগুলো পারমিশন দেখবেন। একটু ধৈর্য নিয়ে সিলেক্ট করুন।

একটা সাধারণ Network Operator-এর জন্য এইগুলো দিতে পারেন:

**DCIM permissions:**
- Can view device
- Can add device
- Can change device
- Can view interface
- Can change interface
- Can view cable
- Can add cable

**IPAM permissions:**
- Can view IP address
- Can add IP address
- Can change IP address
- Can view prefix

মানে তারা ডিভাইস, ইন্টারফেস, ক্যাবল, আইপি দেখতে পারবে আর এডিট করতে পারবে। কিন্তু ডিলিট করতে পারবে না।

**Save** করুন।

এখন ইউজার `jahid`-কে এই গ্রুপে যোগ করুন। Users সেকশনে গিয়ে jahid এডিট করুন। **Groups** ফিল্ডে "Network Operators" সিলেক্ট করুন।

এখন jahid যখন লগইন করবে, সে ডিভাইস দেখতে পারবে, এডিট করতে পারবে, কিন্তু ডিলিট করতে পারবে না।

### আরেকটা Group: Read-Only Users

ধরুন আপনার ম্যানেজমেন্ট টিম নেটওয়ার্ক দেখতে চায় কিন্তু কিছু এডিট করবে না।

```
Name: Read-Only Users

Permissions:
- Can view device
- Can view interface
- Can view IP address
- Can view prefix
- Can view VLAN
- Can view cable
(শুধু view permissions)
```

এই গ্রুপে যারা থাকবে তারা সব দেখতে পারবে কিন্তু কোনো চেঞ্জ করতে পারবে না।

## Quick Tips সমূহ

কিছু টিপস যা আপনার দৈনন্দিন কাজ সহজ করবে:

### ১. Bookmarklet তৈরি করুন

আপনি যদি প্রতিদিন একটা নির্দিষ্ট ফিল্টার করা লিস্ট দেখেন (যেমন আপনার দায়িত্বের সাইটের ডিভাইস), সেই পেজের URL ব্রাউজারে বুকমার্ক করুন। এক ক্লিকে চলে যাবেন।

### ২. Search Syntax শিখুন

Nautobot-এর সার্চ বক্স অনেক স্মার্ট। আপনি লিখতে পারেন:

- `role:core` - সব core role ডিভাইস
- `status:active site:mirpur` - মিরপুরের সব active জিনিস
- `tag:critical` - critical ট্যাগ করা সব কিছু

### ৩. Bulk Edit ব্যবহার করুন

যদি একসাথে অনেকগুলো অবজেক্ট এডিট করতে হয়, লিস্ট ভিউতে চেকবক্স দিয়ে সিলেক্ট করুন। তারপর উপরে **Bulk Edit** অপশন আসবে। একসাথে সব এডিট করতে পারবেন।

### ৪. Export এবং Import Cycle

একটা ভালো প্র্যাক্টিস: মাসে একবার সব ডিভাইসের লিস্ট CSV-তে Export করুন। সেভ করে রাখুন। এটা ব্যাকআপ হিসেবে কাজ করবে। আর যদি কখনো ভুল করে কিছু ডিলিট হয়ে যায়, CSV থেকে Import করে ফেলতে পারবেন।

### ৫. Comments ফিল্ড ম্যাক্সিমাম ব্যবহার করুন

গুরুত্বপূর্ণ যেকোনো তথ্য Comments ফিল্ডে লিখুন। কনফিগ চেঞ্জের হিস্টরি, সমস্যা হওয়ার ইতিহাস, কন্টাক্ট পার্সন - সব। এটা ভবিষ্যতে অমূল্য রেফারেন্স হবে।

## একটা কমপ্লিট উদাহরণ: মিরপুর পপ সম্পূর্ণ সেটআপ

চলুন এখন পর্যন্ত যা শিখলাম সেটা দিয়ে মিরপুর পপের একটা কমপ্লিট পিকচার তৈরি করি।

### ধাপ ১: Provider এবং Circuit

```
Provider: BTCL
Circuit: BTCL-MIR-001 (10 Gbps Fiber)
Connected to: R-MIR-CORE-01, sfpplus1
```

### ধাপ ২: Core Infrastructure

```
Device: R-MIR-CORE-01
Type: MikroTik CCR2004
Role: Core Router
Tags: Production, Critical
Custom Fields:
  - Warranty Expiry: 2027-01-15
  - Purchase Source: International Import
  - Power Consumption: 40W
```

### ধাপ ৩: Distribution Layer

```
Device: SW-MIR-DIST-01
Type: Cisco Catalyst 2960
Role: Distribution Switch
Cable from R-MIR-CORE-01 sfpplus2 to SW-MIR-DIST-01 sfpplus1 (Blue Fiber, 5m)
Tags: Production
```

### ধাপ ৪: Access Layer (3টা সুইচ)

CSV দিয়ে ইমপোর্ট:

```csv
name,device_type,role,location,rack,status,tags
SW-MIR-ACC-01,Cisco Catalyst 2960,Access Switch,Mirpur POP,Rack-A,Active,production
SW-MIR-ACC-02,Cisco Catalyst 2960,Access Switch,Mirpur POP,Rack-A,Active,production
SW-MIR-ACC-03,Cisco Catalyst 2960,Access Switch,Mirpur POP,Rack-A,Active,production
```

### ধাপ ৫: Cable Connections

প্রতিটা Access Switch Core Router বা Distribution Switch-এর সাথে কানেক্টেড:

```
SW-MIR-ACC-01 ether24 <--(Yellow Cat6, 10m)--> SW-MIR-DIST-01 ether1
SW-MIR-ACC-02 ether24 <--(Yellow Cat6, 12m)--> SW-MIR-DIST-01 ether2
SW-MIR-ACC-03 ether24 <--(Yellow Cat6, 15m)--> SW-MIR-DIST-01 ether3
```

### ধাপ ৬: IP Addressing

```
R-MIR-CORE-01:
  - lo0: 10.10.1.1/32 (Loopback/Management)
  - sfpplus1: 103.92.40.2/30 (BTCL Uplink)
  - ether1: 10.10.10.1/24 (Distribution Network)

SW-MIR-DIST-01:
  - vlan10: 10.10.10.2/24 (Management)
```

### ধাপ ৭: VLANs

```
VLAN 10: MANAGEMENT (All network devices)
VLAN 100: RESIDENTIAL_MIRPUR (Customer traffic)
VLAN 200: CORPORATE_MIRPUR (Corporate clients)
```

এই সেটআপ হয়ে গেলে আপনার মিরপুর পপের একটা সম্পূর্ণ ডিজিটাল টুইন তৈরি হয়ে গেছে Nautobot-এ। যে কেউ লগইন করে পুরো নেটওয়ার্ক বুঝতে পারবে।

## কী শিখলাম আমরা?

এই চ্যাপ্টারে আমরা দেখলাম কীভাবে:

১. **Provider এবং Circuit** সেটআপ করতে হয় - কোথা থেকে ইন্টারনেট আসছে সেটা ডকুমেন্ট করা

২. **Cable Management** করতে হয় - কোন তার কোথায় গেছে সেটা ট্র্যাক করা

৩. **Custom Fields** তৈরি করতে হয় - নিজের প্রয়োজন অনুযায়ী ফিল্ড যোগ করা

৪. **Tags** ব্যবহার করতে হয় - সহজে ক্যাটাগরাইজ এবং ফিল্টার করা

৫. **User এবং Permission** ম্যানেজ করতে হয় - টিম মেম্বারদের সঠিক অ্যাক্সেস দেওয়া

এখন আপনার Nautobot সেটআপ প্রায় কমপ্লিট। আপনার নেটওয়ার্কের একটা পরিপূর্ণ ডিজিটাল রিপ্রেজেন্টেশন তৈরি হয়ে গেছে। পরের চ্যাপ্টারে আমরা দেখব রিয়েল ওয়ার্ল্ড ইমপ্লিমেন্টেশন - কীভাবে ছোট আইএসপি থেকে বড় আইএসপিতে স্কেল করবেন, কোন কোন চ্যালেঞ্জ আসবে আর সেগুলোর সমাধান কী।