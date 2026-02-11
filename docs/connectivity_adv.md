আগের চ্যাপ্টারে আমরা বেসিক ডেটা এন্ট্রি শিখলাম - লোকেশন, ডিভাইস, আইপি, ভিল্যান, ক্যাবল। SkyNets Bangladesh এর NOC টিম এখন দৈনন্দিন কাজে Nautobot ব্যবহার শুরু করেছে। কিন্তু কিছুদিন পরেই তারা বুঝতে পারল বেসিক ফিচারগুলো যথেষ্ট না। 

তাদের টিম লিডার জাহাঙ্গীর সাহেব একটা মিটিং ডাকলেন। সমস্যাগুলো শুনলেন:

**আসিফ (NOC টেকনিশিয়ান):** "স্যার, আমাদের ১০০+ ডিভাইস আছে। কোনটার ওয়ারেন্টি কবে শেষ হবে মনে রাখা অসম্ভব। গত সপ্তাহে একটা সুইচ নষ্ট হলো - তখন খুঁজে দেখলাম ওয়ারেন্টি ৬ মাস আগেই শেষ!"

**করিম (ফিল্ড টেকনিশিয়ান):** "আমাকে বলা হয় 'বিল্ডিং সি তে গিয়ে সুইচ চেক করো'। কিন্তু Nautobot এ বিল্ডিং নাম নেই। সব সুইচের নাম SW-DN-MIR-ACC-০১, ০২, ০৩... কোনটা কোথায় বুঝি না!"

**রফিক (ম্যানেজার):** "আমার শুধু ডেটা দেখার দরকার। কিন্তু admin একাউন্ট দিয়ে লগইন করতে হয়। ভুলে কিছু ডিলিট করে দিলে সমস্যা!"

জাহাঙ্গীর সাহেব বললেন, "চিন্তা নেই। Nautobot এ এসবের সমাধান আছে। চলুন দেখি কীভাবে।"

এই চ্যাপ্টারে আমরা Nautobot এর এডভান্সড ফিচারগুলো দেখব যা দিয়ে সিস্টেমকে নিজের মতো কাস্টমাইজ করা যায়।

---

### Custom Fields - নিজস্ব ডেটা ট্র্যাকিং

Nautobot এ অনেক built-in ফিল্ড আছে - name, location, serial number, status। কিন্তু প্রতিটা অর্গানাইজেশনের নিজস্ব কিছু ডেটা ট্র্যাক করার দরকার হয় যা স্ট্যান্ডার্ড ফিল্ডে নেই। এজন্যই Custom Fields।

#### সমস্যা ১: ওয়ারেন্টি ট্র্যাকিং

আসিফের সমস্যাটা solve করি। প্রতিটা ডিভাইসের ওয়ারেন্টি কবে শেষ হবে সেটা ট্র্যাক করতে হবে।

**Extras → Custom Fields** যান। **+ Add** ক্লিক করুন।

একটা ফর্ম আসবে:

```
Label: Warranty Expiry Date
Key: warranty_expiry_date
```

`Label` হলো যেটা UI তে দেখাবে। `Key` হলো ডেটাবেসে যেভাবে স্টোর হবে। সাধারণত Label টাইপ করলে Key অটোমেটিক জেনারেট হয়ে যায়।

```
Group Name: Procurement
```

Group Name দিয়ে related ফিল্ডগুলো একসাথে গ্রুপ করা যায়। এটা অপশনাল কিন্তু organize করার জন্য ভালো।

এখন সবচেয়ে ইম্পর্ট্যান্ট পার্ট:

```
Content Types: dcim | device
```

এখানে বলে দিতে হবে এই ফিল্ড কোথায় দেখাবে। ড্রপডাউনে অনেক অপশন দেখবেন - dcim | device, dcim | rack, ipam | ip address ইত্যাদি। আমরা চাই শুধু ডিভাইসে দেখাক, তাই `dcim | device` সিলেক্ট করুন।

```
Type: Date
```

এটা একটা ডেট ফিল্ড। অন্যান্য টাইপ আছে:
- Text (সাধারণ টেক্সট)
- Integer (পূর্ণসংখ্যা)
- Boolean (হ্যাঁ/না)
- Date (তারিখ)
- URL (ওয়েবসাইট লিংক)
- Selection (ড্রপডাউন সিলেকশন)
- Multiple Selection (একাধিক সিলেক্ট করা যায়)
- JSON (complex ডেটা)

আমাদের ক্ষেত্রে Date ঠিক আছে।

```
Required: No
```

আনচেক রাখুন। মানে সব ডিভাইসে এটা দেওয়া বাধ্যতামূলক না। কারণ কিছু পুরোনো ডিভাইসের ওয়ারেন্টি ইতিমধ্যে শেষ হয়ে গেছে, কিছুতে হয়তো ওয়ারেন্টিই নেই।

```
Default: (খালি রাখুন)

Description: Date when device warranty expires
Help Text: Format: YYYY-MM-DD. Leave blank if no warranty.
```

Description হলো internal - এটা admin দের জন্য। Help Text হলো যেটা ইউজারকে গাইড করবে ফিল্ড পূরণ করার সময়।

```
Weight: 100
```

Weight যত কম, ফিল্ড তত উপরে দেখাবে। ডিফল্ট 100 ঠিক আছে।

এখন **Create** ক্লিক করুন।

Custom Field তৈরি হয়ে গেছে! এখন যেকোনো ডিভাইস এডিট করলে এই নতুন ফিল্ড দেখবেন।

চলুন মিরপুর কোর রাউটারে যোগ করি:

**Devices → Devices → R-DN-MIR-CORE-01** এ যান। **Edit** বাটনে ক্লিক করুন।

নিচের দিকে স্ক্রল করুন। **Custom Fields** নামে একটা সেকশন দেখবেন:

```
Procurement
  └── Warranty Expiry Date: [        ]
```

একটা ডেট পিকার দেখাবে। ক্লিক করুন এবং তারিখ সিলেক্ট করুন। ধরুন এই রাউটার কেনা হয়েছিল ডিসেম্বর ২০২২ এ, ৩ বছরের ওয়ারেন্টি। তাহলে:

```
Warranty Expiry Date: 2025-12-31
```

**Update** বাটনে ক্লিক করুন।

এখন ডিভাইস ডিটেইল পেজে দেখবেন:

```
Custom Fields
  Warranty Expiry Date: 2025-12-31
```

চমৎকার! এখন সব ডিভাইসে ওয়ারেন্টি ডেট যোগ করুন:

```
SW-DN-MIR-DIST-01: 2026-06-15
SW-DN-MIR-ACC-01: 2025-03-20
SW-DN-MIR-ACC-02: 2025-03-20
```

**এখন ম্যাজিক দেখুন - ফিল্টারিং:**

কিছুদিন পরে জাহাঙ্গীর সাহেব বললেন, "আমাকে একটা লিস্ট দাও - পরের তিন মাসে কোন ডিভাইসগুলোর ওয়ারেন্টি শেষ হবে। সেগুলোর জন্য AMC (Annual Maintenance Contract) রিনিউ করতে হবে।"

আসিফ Nautobot খুলল:

**Devices → Devices** লিস্টে গেল। ডান দিকে **Filters** প্যানেল খুলল।

নিচের দিকে স্ক্রল করলে **Custom Fields** সেকশন দেখবে:

```
Warranty Expiry Date
  ├─ After: [        ]
  └─ Before: [        ]
```

আজকের তারিখ ২০২৫-০২-০৯। তিন মাস পরে ২০২৫-০৫-০৯।

```
After: 2025-02-09
Before: 2025-05-09
```

**Apply** করল।

লিস্টে শুধু সেই ডিভাইসগুলো দেখাল যাদের ওয়ারেন্টি পরের তিন মাসে শেষ হবে:

```
SW-DN-MIR-ACC-01 - Warranty: 2025-03-20 (৪০ দিন বাকি)
```

আসিফ এই লিস্ট **Export** বাটনে ক্লিক করে CSV ডাউনলোড করল। জাহাঙ্গীর সাহেবকে মেইল করে দিল। সমস্যা solved!

#### সমস্যা ২: বিল্ডিং নাম ট্র্যাকিং

এখন করিমের সমস্যা solve করি। এক্সেস সুইচগুলো বিভিন্ন বিল্ডিংয়ে আছে। লোকেশন ফিল্ডে "Mirpur POP" আছে, কিন্তু exact বিল্ডিং নাম নেই।

আরেকটা Custom Field তৈরি করি:

**Extras → Custom Fields → + Add**

```
Label: Building Name
Key: building_name
Group Name: Location Details

Content Types: dcim | device

Type: Text

Required: No
Default: (খালি)

Description: Specific building or area where device is located
Help Text: e.g., "Building A, Sector 12" or "Metro Tower, Floor 5"

Validation Regex: (খালি রাখুন)
```

Validation Regex দিয়ে pattern চেক করানো যায়। যেমন শুধু "Building X" ফরম্যাট allow করতে চাইলে regex দিতে পারেন। আমাদের দরকার নেই এখন।

```
Weight: 110
```

**Create** করুন।

এখন এক্সেস সুইচগুলোতে বিল্ডিং নাম যোগ করুন:

```
SW-DN-MIR-ACC-01 → Edit
  Building Name: Building A, Sector 12

SW-DN-MIR-ACC-02 → Edit
  Building Name: Building B, Sector 12

SW-DN-MIR-ACC-03 → Edit
  Building Name: Building C, Sector 13
```

এখন করিমকে যদি বলা হয় "Building C তে যাও", সে Nautobot এ সার্চ করতে পারবে:

**Devices → Devices** লিস্টে গিয়ে উপরের সার্চ বক্সে টাইপ করবে:

```
Building C
```

গ্লোবাল সার্চ সব ফিল্ডে খুঁজবে, Custom Fields সহ। SW-DN-MIR-ACC-03 খুঁজে পাবে।

অথবা ফিল্টার ব্যবহার করতে পারে:

```
Filters প্যানেল → Custom Fields → Building Name
টাইপ করুন: Building C
```

শুধু সেই সুইচ দেখাবে।

#### আরো কিছু Useful Custom Fields

SkyNets Bangladesh আরো কয়েকটা Custom Field তৈরি করেছে:

**পাওয়ার কনজাম্পশন:**

```
Label: Power Consumption (Watts)
Content Types: dcim | device
Type: Integer
Validation Minimum: 0
Validation Maximum: 10000
Description: Device power consumption in watts
```

এখন র্যাকের টোটাল পাওয়ার ক্যালকুলেট করা যায়। Rack A তে যদি ৫টা ডিভাইস থাকে:

```
R-DN-MIR-CORE-01: 85W
SW-DN-MIR-DIST-01: 45W
SW-DN-MIR-ACC-01: 25W
SW-DN-MIR-ACC-02: 25W
SW-DN-MIR-ACC-03: 25W

Total: 205W
```

UPS সাইজ করার জন্য এই ডেটা কাজে লাগে।

**পারচেজ সোর্স:**

```
Label: Purchase Source
Content Types: dcim | device
Type: Selection
Choices:
  Local Vendor
  International Import
  Rental
  Donation
  Second Hand
Description: How this device was procured
```

এখন রিপোর্ট করতে পারবেন: "আমাদের ৬০% ডিভাইস লোকাল ভেন্ডর থেকে, ৩০% ইমপোর্ট, ১০% রেন্টাল।"

**কাস্টমার কাউন্ট (এক্সেস সুইচের জন্য):**

```
Label: Approximate Customers
Content Types: dcim | device
Type: Integer
Description: Approximate number of customers served by this device
```

```
SW-DN-MIR-ACC-01: 280 customers
SW-DN-MIR-ACC-02: 310 customers
SW-DN-MIR-ACC-03: 265 customers
```

এখন জানেন কোন সুইচে বেশি লোড আছে।

---

### Tags - সহজ ক্যাটেগরাইজেশন

ট্যাগ হলো ডেটা ক্যাটেগরাইজ করার একটা সহজ উপায়। যেকোনো অবজেক্টে একাধিক ট্যাগ যোগ করা যায়।

#### কেন ট্যাগ দরকার?

ধরুন আপনি জানতে চান:
- কোন ডিভাইসগুলো প্রোডাকশনে আছে?
- কোন ডিভাইসগুলো ক্রিটিক্যাল (যেগুলো down হলে বড় সমস্যা)?
- কোনগুলো এখন মেইনটেনেন্সে আছে?

এসবের জন্য আলাদা আলাদা ফিল্ড তৈরি করা impractical। ট্যাগ দিয়ে সহজেই করা যায়।

#### ট্যাগ তৈরি করা

**Extras → Tags** যান। **+ Add** ক্লিক করুন।

**Production Tag:**

```
Name: production
Color: 4caf50
```

Color একটা হেক্স কোড। অথবা Color Picker ব্যবহার করুন। সবুজ রঙ মানে "সব ঠিক আছে, চলছে"।

```
Description: Production/live equipment actively serving customers

Content Types: (খালি রাখুন)
```

Content Types খালি রাখলে সব ধরনের অবজেক্টে এই ট্যাগ ব্যবহার করা যাবে - ডিভাইস, আইপি, সার্কিট, সব জায়গায়।

**Create and Add Another** ক্লিক করুন (যদি আরো ট্যাগ তৈরি করতে চান)।

একইভাবে আরো কয়েকটা:

**Critical Tag:**

```
Name: critical
Color: f44336 (লাল)
Description: Critical infrastructure - failure causes major outage
```

**Backup Tag:**

```
Name: backup
Color: 2196f3 (নীল)
Description: Backup/standby equipment
```

**End of Life Tag:**

```
Name: end-of-life
Color: 9e9e9e (ধূসর)
Description: Equipment past end of life, replacement needed
```

**Maintenance Tag:**

```
Name: maintenance
Color: ffeb3b (হলুদ)
Description: Currently under maintenance
```

এখন **Extras → Tags** লিস্টে পাঁচটা কালারফুল ট্যাগ দেখবেন।

#### ডিভাইসে ট্যাগ যোগ করা

**Devices → Devices → R-DN-MIR-CORE-01 → Edit**

```
Tags: (এখানে মাল্টিপল সিলেক্ট করা যায়)
  - production
  - critical
```

দুটো ট্যাগ সিলেক্ট করুন (Ctrl চেপে রাখুন বা ড্রপডাউনে ক্লিক করুন)।

**Update** করুন।

ডিভাইস ডিটেইল পেজে এখন দুটো কালারফুল ব্যাজ দেখবেন:

```
Tags: [production] [critical]
```

সবুজ এবং লাল ব্যাজ।

একইভাবে অন্যান্য ডিভাইসে:

```
SW-DN-MIR-DIST-01
  Tags: production

SW-DN-MIR-ACC-01
  Tags: production

R-DN-MIR-CORE-02 (যদি ব্যাকআপ রাউটার থাকে)
  Tags: backup
```

#### ট্যাগ দিয়ে ফিল্টারিং

**Devices → Devices** লিস্টে যান।

**Filters** প্যানেল খুলুন:

```
Tags: critical
```

শুধু critical ট্যাগ ওয়ালা ডিভাইস দেখাবে। SkyNets এর ক্ষেত্রে:

```
R-DN-MIR-CORE-01
R-DN-UTT-CORE-01
```

দুটো কোর রাউটার।

মাল্টিপল ট্যাগ দিয়েও ফিল্টার করা যায়:

```
Tags: production, critical
```

যেসব ডিভাইসে **উভয়** ট্যাগ আছে শুধু সেগুলো দেখাবে।

#### ট্যাগ এর প্র্যাক্টিক্যাল ব্যবহার

**Scenario 1: মেইনটেনেন্স উইন্ডো**

মিরপুর কোর রাউটারে একটা ফার্মওয়্যার আপগ্রেড করতে হবে। রাত ২টায় কাজ শুরু হবে, ৪টায় শেষ।

কাজ শুরুর আগে:

```
R-DN-MIR-CORE-01 → Edit
  Status: Offline (Active থেকে চেঞ্জ করুন)
  Tags: maintenance যোগ করুন
  Comments: "Firmware upgrade in progress. Scheduled: 2025-02-10 02:00-04:00 AM. Contact: Asif +880 1712-345678"
```

এখন যে কেউ Nautobot দেখলে বুঝবে এই রাউটার মেইনটেনেন্সে আছে।

কাজ শেষ হলে:

```
Status: Active
Tags: maintenance রিমুভ করুন
Comments: "Firmware upgraded to v7.12. Completed: 2025-02-10 03:45 AM"
```

**Scenario 2: EOL (End of Life) ট্র্যাকিং**

কিছু পুরোনো সুইচ আছে যেগুলো ৫ বছর হয়ে গেছে, রিপ্লেস করা দরকার:

```
SW-DN-MIR-ACC-05 → Edit
  Tags: end-of-life যোগ করুন
  Comments: "Purchased 2020. Frequent issues. Replacement budgeted for Q2 2025."
```

এখন বাজেট মিটিংয়ে:

```
Filters → Tags: end-of-life
```

সব EOL ডিভাইসের লিস্ট। Export করে Excel এ নিন, টোটাল রিপ্লেসমেন্ট কস্ট ক্যালকুলেট করুন।

---

### Saved Filters - কমন কোয়েরি সেভ করা

আসিফ প্রতিদিন সকালে Nautobot এ যায় এবং মিরপুর পপের সব এক্টিভ ডিভাইস চেক করে। প্রতিদিন একই ফিল্টার:

```
Location: Mirpur POP
Status: Active
```

এটা প্রতিদিন সেট করা বিরক্তিকর। Saved Filter দিয়ে সহজ করা যায়।

#### একটা Saved Filter তৈরি করা

**Devices → Devices** লিস্টে যান।

ফিল্টার সেট করুন:

```
Location: Mirpur POP
Status: Active
```

**Apply** করুন। লিস্টে শুধু মিরপুরের এক্টিভ ডিভাইস দেখাবে।

এখন উপরে ডান দিকে **Save Filter** বাটন দেখবেন। ক্লিক করুন।

একটা ছোট ডায়ালগ বক্স:

```
Name: Mirpur Active Devices
Public: ☐ (আনচেক রাখুন)
```

Public চেক করলে অন্য ইউজাররাও এই ফিল্টার দেখবে। আনচেক থাকলে শুধু আপনি দেখবেন।

**Save** ক্লিক করুন।

এখন পরের বার **Devices → Devices** লিস্টে গেলে উপরে **Saved Filters** নামে একটা ড্রপডাউন দেখবেন। ক্লিক করলে:

```
Saved Filters
  └── Mirpur Active Devices
```

এটা সিলেক্ট করলেই ফিল্টার অটোমেটিক্যালি এপ্লাই হয়ে যাবে। এক ক্লিকে কাজ শেষ!

#### আরো কিছু Useful Saved Filters

**Critical Infrastructure:**

```
Tags: critical
Name: Critical Infrastructure Only
Public: Yes (সবাই দেখুক)
```

**Warranty Expiring Soon:**

```
Warranty Expiry Date:
  After: 2025-02-09 (আজ)
  Before: 2025-05-09 (৯০ দিন পরে)
Name: Warranty Expiring in 90 Days
```

**Access Switches - All Locations:**

```
Role: Access Switch
Status: Active
Name: All Active Access Switches
```

**Devices Under Maintenance:**

```
Tags: maintenance
Name: Currently Under Maintenance
```

এভাবে আপনার frequent queries সেভ করে রাখুন। সময় বাঁচবে এবং consistency থাকবে।

---

### User Management এবং Permissions

SkyNets Bangladesh এর তিন ধরনের ইউজার আছে:

১. **NOC Team** (জাহাঙ্গীর, আসিফ, রহিম): সবকিছু দেখতে এবং এডিট করতে পারবে
২. **Field Operations** (করিম, সালমান): শুধু ডিভাইস এবং ক্যাবল ম্যানেজ করবে
৩. **Management** (রফিক সাহেব - CTO): শুধু রিপোর্ট দেখবে, কিছু এডিট করবে না

প্রতিটা টিমের জন্য আলাদা permission লেভেল দরকার।

#### Groups তৈরি করা

Nautobot এ Groups দিয়ে permissions ম্যানেজ করা হয়। একটা গ্রুপে permissions সেট করুন, তারপর ইউজারদের সেই গ্রুপে যোগ করুন।

উপরের ডান কোণায় আপনার ইউজারনেম (admin) ক্লিক করুন → **Admin** সিলেক্ট করুন।

Django Admin প্যানেলে চলে যাবেন। এখানে **Groups** দেখবেন। ক্লিক করুন।

**+ Add Group** বাটনে ক্লিক করুন।

**Group 1: NOC Team**

```
Name: NOC Team
```

এখন Permissions সিলেক্ট করতে হবে। নিচে একটা বড় বক্স দেখবেন **Available permissions** এবং **Chosen permissions**।

বাম দিকে (Available) থেকে ডান দিকে (Chosen) মুভ করতে হবে যেগুলো দরকার।

এটা একটু সময় সাপেক্ষ কারণ অনেক permissions আছে। আমরা ক্যাটেগরি অনুযায়ী দেখব:

**DCIM - Device Permissions:**

```
dcim | device | Can view device
dcim | device | Can add device
dcim | device | Can change device
```

লক্ষ্য করুন "Can delete device" সিলেক্ট করছি না। NOC ডিভাইস ডিলিট করতে পারবে না (ভুল করে ডিলিট হয়ে যাওয়ার ভয়)।

**DCIM - Cable Permissions:**

```
dcim | cable | Can view cable
dcim | cable | Can add cable
dcim | cable | Can change cable
```

**DCIM - Interface Permissions:**

```
dcim | interface | Can view interface
dcim | interface | Can change interface
```

**IPAM - IP Address Permissions:**

```
ipam | ip address | Can view IP address
ipam | ip address | Can add IP address
ipam | ip address | Can change IP address
```

**IPAM - Prefix Permissions:**

```
ipam | prefix | Can view prefix
ipam | prefix | Can add prefix
ipam | prefix | Can change prefix
```

**Circuits - Limited Access:**

```
circuits | circuit | Can view circuit
```

শুধু view, add/change/delete না। কারণ সার্কিট যোগ/পরিবর্তন করা ম্যানেজমেন্ট লেভেলের কাজ।

**Organization - Location (View Only):**

```
Organization | location | Can view location
```

NOC লোকেশন দেখবে কিন্তু নতুন লোকেশন যোগ বা মুছবে না।

এভাবে সব relevant permissions সিলেক্ট করুন। তারপর **Save** ক্লিক করুন।

**Group 2: Field Operations**

```
Name: Field Operations
```

**Permissions:**

```
dcim | device | Can view device
dcim | device | Can add device (নতুন এক্সেস সুইচ ইনস্টল করার পরে যোগ করবে)
dcim | device | Can change device (Comments আপডেট করবে)

dcim | cable | Can view cable
dcim | cable | Can add cable
dcim | cable | Can change cable

dcim | interface | Can view interface
dcim | interface | Can change interface (পোর্ট enable/disable করবে)

ipam | ip address | Can view IP address
```

লক্ষ্য করুন - IP address শুধু view, add/change না। ফিল্ড টিম আইপি এলোকেশন করবে না, NOC করবে।

**Save** করুন।

**Group 3: Management**

```
Name: Management
```

**Permissions:**

সব জায়গায় শুধু **Can view** সিলেক্ট করুন। কোনো add/change/delete না।

```
dcim | device | Can view device
dcim | cable | Can view cable
dcim | interface | Can view interface
dcim | rack | Can view rack
ipam | ip address | Can view IP address
ipam | prefix | Can view prefix
ipam | vlan | Can view vlan
circuits | circuit | Can view circuit
circuits | provider | Can view provider
Organization | location | Can view location
```

**Save** করুন।

#### নতুন ইউজার তৈরি করা

এখন actual users তৈরি করি।

Django Admin প্যানেলে **Users** এ যান। **+ Add User** ক্লিক করুন।

**ইউজার: আসিফ (NOC Team)**

```
Username: asif
Password: (একটা শক্তিশালী পাসওয়ার্ড দিন)
Password confirmation: (আবার টাইপ করুন)
```

**Save and continue editing** ক্লিক করুন।

এখন আরো ফিল্ড দেখাবে:

```
Personal Info:
  First name: Asif
  Last name: Rahman
  Email address: asif@skynets.bd

Permissions:
  Active: ☑ (চেক করুন - না হলে লগইন করতে পারবে না)
  Staff status: ☑ (চেক করুন - এটা থাকলে Nautobot admin প্যানেলে access পাবে)
  Superuser status: ☐ (আনচেক - superuser শুধু আপনি থাকবেন)

Groups:
  NOC Team (এখানে সিলেক্ট করুন)
```

Groups সেকশনে Available groups থেকে "NOC Team" সিলেক্ট করে ডান দিকে মুভ করুন।

**Save** ক্লিক করুন।

একইভাবে আরো দুজন:

**করিম (Field Operations):**

```
Username: karim
First name: Karim
Last name: Uddin
Email: karim@skynets.bd
Active: ☑
Staff status: ☑
Groups: Field Operations
```

**রফিক সাহেব (Management):**

```
Username: rafiq
First name: Rafiq
Last name: Ahmed
Email: rafiq@skynets.bd
Active: ☑
Staff status: ☑
Groups: Management
```

#### Permissions টেস্ট করা

এখন লগআউট করুন admin থেকে। asif দিয়ে লগইন করুন:

```
Username: asif
Password: (যেটা দিয়েছিলেন)
```

লগইন করার পরে Nautobot হোম পেজে আসবেন।

**Devices → Devices** লিস্টে যান।

একটা ডিভাইস ক্লিক করুন - দেখতে পারছেন।

**Edit** বাটন আছে - ক্লিক করুন, এডিট করতে পারছেন।

কিন্তু **Delete** বাটন নেই! Permission নেই বলে।

**Organization → Locations** যান।

লিস্ট দেখতে পারছেন। কিন্তু **+ Add** বাটন নেই। নতুন লোকেশন যোগ করতে পারবেন না।

পারফেক্ট! Permission কাজ করছে।

এখন লগআউট করুন। rafiq দিয়ে লগইন করুন (ম্যানেজমেন্ট গ্রুপ):

```
Username: rafiq
Password: ...
```

লগইন করার পরে:

**Devices → Devices** - লিস্ট দেখতে পারছেন।

কিন্তু কোথাও **Edit**, **Delete**, **+ Add** কোনো বাটন নেই!

সম্পূর্ণ রিড-অনলি। রফিক সাহেব শুধু রিপোর্ট দেখতে পারবেন, ভুল করে কিছু চেঞ্জ করতে পারবেন না।

চমৎকার!

---

SkyNets Bangladesh এখন Nautobot কে তাদের নিজস্ব প্রয়োজন অনুযায়ী সাজিয়ে ফেলেছে। আসিফের ওয়ারেন্টি ট্র্যাকিং সমস্যা solve হয়েছে। করিম এখন সহজেই বিল্ডিং নাম দিয়ে সুইচ খুঁজে পায়। রফিক সাহেব নিরাপদে রিপোর্ট দেখতে পারেন ভুলে কিছু চেঞ্জ হওয়ার ভয় ছাড়া।

পরের চ্যাপ্টারে আমরা দেখব কীভাবে ৮ হাজার কাস্টমার থেকে ৫০ হাজার কাস্টমারে স্কেল করা যায়, কী কী চ্যালেঞ্জ আসে, এবং কীভাবে Nautobot সেই জার্নিতে সাহায্য করে।

---