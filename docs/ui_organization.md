আগের চ্যাপ্টারে আমরা Nautobot 3.0 এর ডেটা মডেল বুঝলাম - Organization, DCIM, IPAM, Circuits, Extras। এখন সময় এসেছে হাতে-কলমে কাজ শুরু করার। এই চ্যাপ্টারে আমরা দেখব কীভাবে Nautobot এর ওয়েব UI ব্যবহার করে actual ডেটা এন্ট্রি করতে হয়।

Nirvor Communication এর NOC টিম এখন তাদের মিরপুর এবং উত্তরা পপের সম্পূর্ণ নেটওয়ার্ক Nautobot এ ডকুমেন্ট করবে। তাদের সাথে একসাথে আমরাও শিখব। শুরু করা যাক।

### Nautobot 3.0 UI Overview

প্রথমে একবার Nautobot UI এর সাথে পরিচিত হয়ে নিই।

#### লগইন এবং ড্যাশবোর্ড

ব্রাউজারে যান:

```
https://nautobot.nirvor.bd
```

লগইন করুন:
```
Username: admin
Password: Nirvor@Admin2025!
```

লগইন করার পরে আপনি ড্যাশবোর্ডে আসবেন। প্রথমবার লগইন করলে ড্যাশবোর্ড খালি দেখাবে - এটা স্বাভাবিক। এখনও কোনো ডেটা নেই।

#### নেভিগেশন বার

উপরে একটা নেভিগেশন বার দেখবেন। এখানে মূল মেনুগুলো:

**Organization:** Locations, Teams, Tenants, Contacts

**Devices:** Device Types, Manufacturers, Devices, Racks, Cables, Device Redundancy Groups

**IPAM:** Namespaces, Prefixes, IP Addresses, VLANs, VLAN Groups, RIRs

**Circuits:** Providers, Circuit Types, Circuits

**Power:** Power Feeds, Power Panels (আমরা এগুলো ব্যবহার করব না এই বইতে)

**Extras:** Roles, Statuses, Tags, Custom Fields, Relationships, Jobs, Git Repositories

**Plugins:** (যদি কোনো plugin ইনস্টল করা থাকে)

#### সার্চ ফিচার

ডান দিকে উপরের কোণায় একটা সার্চ বক্স দেখবেন। এটা গ্লোবাল সার্চ - যেকোনো কিছু এখানে টাইপ করলে Nautobot সব জায়গায় খুঁজবে।

টেস্ট করুন - টাইপ করুন "mirpur" (এখনও কিছু পাবেন না কারণ ডেটা নেই)।

#### ইউজার মেনু

একদম ডান দিকে উপরের কোণায় আপনার ইউজারনেম দেখবেন। ক্লিক করলে একটা ড্রপডাউন:

- **Profile:** আপনার প্রোফাইল দেখুন/এডিট করুন
- **Preferences:** UI সেটিংস (items per page, theme ইত্যাদি)
- **API Tokens:** API টোকেন ম্যানেজ করুন
- **Change Password:** পাসওয়ার্ড চেঞ্জ করুন
- **Logout:** লগআউট

---

### লোকেশন হায়ারার্কি তৈরি করা

প্রথম কাজ হলো Location Hierarchy সেটআপ করা। এটা Nautobot 3.0 এর সবচেয়ে ইম্পর্ট্যান্ট ফিচার।

#### স্টেপ ১: Location Types ডিফাইন করা

**Organization → Location Types** যান।

খালি পেজ দেখাবে। ডান দিকে উপরে **+ Add** বাটনে ক্লিক করুন।

**প্রথম Location Type: Zone**

```
Name: Zone
Description: Geographic coverage zone
Nestable: ✓ (চেক করুন)
Content Types: (খালি রাখুন - সব কিছুতে ইউজ হতে পারে)
```

**Create** ক্লিক করুন।

এখন আরেকটা - **+ Add** আবার ক্লিক করুন।

**দ্বিতীয় Location Type: Cluster**

```
Name: Cluster
Description: Group of nearby POPs for operational management
Nestable: ✓
Parent Location Type: Zone (এটা ড্রপডাউন থেকে সিলেক্ট করুন)
```

**Create** করুন।

**তৃতীয় Location Type: POP**

```
Name: POP
Description: Point of Presence - ISP network node
Nestable: ✓
Parent Location Type: Cluster
```

**চতুর্থ Location Type: Rack**

```
Name: Rack
Description: Equipment rack
Nestable: ✗ (আনচেক রাখুন - এর নিচে আর কিছু হবে না)
Parent Location Type: POP
```

এখন আপনার চারটা Location Type তৈরি হয়েছে। **Organization → Location Types** এ গিয়ে দেখুন - সব লিস্টেড দেখাবে।

#### স্টেপ ২: Actual Locations তৈরি করা

এখন real locations তৈরি করব।

**Organization → Locations** যান। **+ Add** ক্লিক করুন।

**Top Level Location: Dhaka North Zone**

```
Name: Dhaka North Zone
Location Type: Zone (ড্রপডাউন থেকে সিলেক্ট করুন)
Status: Active (ড্রপডাউন থেকে)
Parent Location: (খালি রাখুন - এটা top level)
Tenant: (খালি রাখুন)
Description: Northern Dhaka metropolitan area coverage
```

**Create** ক্লিক করুন।

এখন ভেতরে ঢুকুন - **Mirpur Cluster** তৈরি করব।

আবার **Organization → Locations → + Add** যান।

```
Name: Mirpur Cluster
Location Type: Cluster
Status: Active
Parent Location: Dhaka North Zone (এটা খুব ইম্পর্ট্যান্ট!)
Description: Mirpur and Kalyanpur area operations cluster
```

**Create** করুন।

এবার Mirpur POP:

```
Name: Mirpur POP
Location Type: POP
Status: Active
Parent Location: Mirpur Cluster
Facility: Mirpur-12 Data Center
Physical Address: House 25, Road 10, Mirpur-12, Dhaka-1216, Bangladesh
Shipping Address: (Same as physical address)
Latitude: 23.8103
Longitude: 90.3654
Contact Name: Jahangir Khan
Contact Phone: +880 1712-345678
Contact Email: jahangir@nirvor.bd
Time Zone: Asia/Dhaka
Description: Primary POP serving Mirpur area. Approximately 4000 customers.
Comments: 
  - Established: January 2023
  - Building: 3-story commercial building
  - Power: Dual utility feed + 50 KVA generator backup
  - Cooling: 2x 5-ton AC units
```

**Create** করুন।

**GPS Coordinates কীভাবে পাবেন?**

Google Maps এ যান → আপনার লোকেশন সার্চ করুন → রাইট ক্লিক → "What's here?" → নিচে coordinates দেখাবে → কপি করুন।

এখন একইভাবে Uttara Cluster এবং Uttara POP তৈরি করুন:

**Uttara Cluster:**

```
Name: Uttara Cluster
Location Type: Cluster
Status: Active
Parent Location: Dhaka North Zone
Description: Uttara and Baridhara area operations cluster
```

**Uttara POP:**

```
Name: Uttara POP
Location Type: POP
Status: Active
Parent Location: Uttara Cluster
Facility: Uttara Sector-7 Office
Physical Address: Plot 15, Road 12, Sector-7, Uttara, Dhaka-1230, Bangladesh
Latitude: 23.8759
Longitude: 90.3795
Contact Name: Asif Rahman
Contact Phone: +880 1811-234567
Contact Email: asif@nirvor.bd
Time Zone: Asia/Dhaka
Description: Secondary POP serving Uttara area. Approximately 4000 customers.
```

এখন **Organization → Locations** এ যান। আপনার লোকেশন হায়ারার্কি দেখুন:

```
Dhaka North Zone
  ├── Mirpur Cluster
  │    └── Mirpur POP
  └── Uttara Cluster
       └── Uttara POP
```

চমৎকার! লোকেশন স্ট্রাকচার রেডি।

---

### Unified Roles সেটআপ

Nautobot 3.0 এর একটা বড় চেঞ্জ হলো Unified Roles। এখন **Extras → Roles** এ সব roles থাকবে।

#### ডিভাইস Roles তৈরি করা

**Extras → Roles** যান। **+ Add** ক্লিক করুন।

**প্রথম Role: Core Router**

```
Name: Core Router
Content Types: dcim | device (এটা খুব ইম্পর্ট্যান্ট - এই dropdown থেকে সিলেক্ট করুন)
Color: f44336 (লাল রঙের হেক্স কোড)
Weight: 1000 (যত বেশি, তত important)
Description: Core routing equipment at the heart of the network
```

**Create** করুন।

কালার কীভাবে বেছে নেবেন? Color picker দেখাবে অথবা hex code সরাসরি দিতে পারেন:
- Red: f44336
- Orange: ff9800
- Green: 4caf50
- Blue: 2196f3
- Purple: 9c27b0

এখন আরো কয়েকটা role তৈরি করুন:

**Distribution Switch:**

```
Name: Distribution Switch
Content Types: dcim | device
Color: ff9800 (Orange)
Weight: 800
Description: Distribution layer switches aggregating access switches
```

**Access Switch:**

```
Name: Access Switch
Content Types: dcim | device
Color: 4caf50 (Green)
Weight: 600
Description: Access layer switches connecting to customers
```

**Firewall:**

```
Name: Firewall
Content Types: dcim | device
Color: 9c27b0 (Purple)
Weight: 950
Description: Security devices and firewalls
```

এখন **Extras → Roles** এ গিয়ে দেখুন - চারটা role দেখাবে কালারফুল badges সহ।

---

### Manufacturers এবং Device Types

ডিভাইস যোগ করার আগে তাদের টাইপ ডিফাইন করতে হবে।

#### Manufacturers যোগ করা

**Devices → Manufacturers** যান। **+ Add** ক্লিক করুন।

**MikroTik:**

```
Name: MikroTik
Description: Latvian network equipment manufacturer specializing in routers and wireless systems
```

**Create** করুন।

একইভাবে আরো:

**TP-Link:**

```
Name: TP-Link
Description: Chinese networking equipment manufacturer
```

**Cisco:**

```
Name: Cisco
Description: American multinational technology company - networking equipment
```

#### Device Types তৈরি করা

এখন ডিভাইস টাইপ বানাবেন। এটা একটু সময় নেয় কারণ interface templates যোগ করতে হয়।

**Devices → Device Types** যান। **+ Add** ক্লিক করুন।

**MikroTik CCR2004:**

```
Manufacturer: MikroTik (ড্রপডাউন থেকে)
Model: CCR2004-1G-12S+2XS
Part Number: CCR2004-1G-12S+2XS
Height (U): 1
Is Full Depth: ✓
Subdevice Role: None
Description: Cloud Core Router with 1x 1G, 12x 10G SFP+, 2x 25G SFP28 ports
Comments: Typical use - Core router for small to medium ISPs
```

**Create** ক্লিক করুন।

এখন ইন্টারফেস টেমপ্লেট যোগ করতে হবে। Device Type এর ডিটেইল পেজে আসবেন।

**Interface Templates** ট্যাবে ক্লিক করুন। **+ Add Interface Templates** বাটনে ক্লিক করুন।

একসাথে একাধিক ইন্টারফেস যোগ করতে পারবেন:

**Management Interface:**

```
Name: ether1
Type: 1000BASE-T (1GE)
Management Only: ✓ (চেক করুন - এটা ম্যানেজমেন্ট পোর্ট)
```

**Create and Add Another** ক্লিক করুন।

**SFP+ Ports (একটা একটা করে বা pattern ব্যবহার করে):**

Nautobot smart - আপনি pattern ব্যবহার করতে পারেন:

```
Name: sfp-sfpplus[1-12]
Type: 10GBASE-X-SFP+
Management Only: (আনচেক)
```

এটা করলে sfp-sfpplus1 থেকে sfp-sfpplus12 পর্যন্ত ১২টা ইন্টারফেস একসাথে তৈরি হবে।

**Create and Add Another** ক্লিক করুন।

**SFP28 Ports:**

```
Name: sfp28-[1-2]
Type: 25GBASE-X-SFP28
Management Only: (আনচেক)
```

এখন **Create** ক্লিক করুন (শেষ যেহেতু)।

Interface Templates ট্যাবে গিয়ে দেখুন - ১৫টা ইন্টারফেস দেখাবে (১ + ১২ + ২)।

চমৎকার! এখন যখনই এই ডিভাইস টাইপের একটা নতুন ডিভাইস তৈরি করবেন, সব ইন্টারফেস অটোমেটিক্যালি কপি হয়ে যাবে।

একইভাবে আরো দুটো ডিভাইস টাইপ তৈরি করুন:

**TP-Link TL-SG3428:**

```
Manufacturer: TP-Link
Model: TL-SG3428
Height (U): 1
Is Full Depth: ✓
Description: 28-Port Gigabit L2+ Managed Switch with 4 SFP slots

Interface Templates:
  - ether[1-24]: Type 1000BASE-T
  - sfp[1-4]: Type 1000BASE-X-SFP
```

**TP-Link TL-SG1024D:**

```
Manufacturer: TP-Link
Model: TL-SG1024D
Height (U): 1
Is Full Depth: ✓
Description: 24-Port Gigabit Unmanaged Switch

Interface Templates:
  - ether[1-24]: Type 1000BASE-T
```

---

### Racks তৈরি করা

এখন র‍্যাক যোগ করব যেখানে ডিভাইস মাউন্ট করা হবে।

**Devices → Racks** যান। **+ Add** ক্লিক করুন।

**Mirpur POP Rack A:**

```
Name: Rack A
Location: Mirpur POP (ড্রপডাউন থেকে সিলেক্ট করুন - এটা hierarchy তে দেখাবে)
Status: Active
Facility ID: MIR-RACK-A
Type: 4-post cabinet
Width: 19 inches
Height (U): 42
Outer Width: 600 (mm)
Outer Depth: 1000 (mm)
Description: Primary equipment rack in Mirpur POP
Comments: Rack installed January 2023. APC NetShelter SV 42U.
```

**Create and Add Another** ক্লিক করুন (যদি আরো র‍্যাক যোগ করতে চান)।

**Mirpur POP Rack B:**

```
Name: Rack B
Location: Mirpur POP
Status: Active
Facility ID: MIR-RACK-B
Type: 4-post cabinet
Width: 19 inches
Height (U): 42
Description: Secondary equipment rack in Mirpur POP
```

**Create** ক্লিক করুন।

একইভাবে Uttara POP এর জন্য:

**Uttara POP Rack A:**

```
Name: Rack A
Location: Uttara POP
Status: Active
Facility ID: UTT-RACK-A
Type: 4-post cabinet
Width: 19 inches
Height (U): 42
Description: Primary equipment rack in Uttara POP
```

এখন **Devices → Racks** লিস্টে গিয়ে দেখুন - তিনটা র‍্যাক দেখাবে।

---

### প্রথম Device এন্ট্রি - মিরপুর কোর রাউটার

এখন সবচেয়ে exciting অংশ - প্রথম ডিভাইস যোগ করা!

**Devices → Devices** যান। **+ Add** ক্লিক করুন।

একটা বড় ফর্ম আসবে। ঘাবড়াবেন না - ধাপে ধাপে পূরণ করব।

```
Name: R-DN-MIR-CORE-01
Device Type: MikroTik CCR2004-1G-12S+2XS (টাইপ করলে autocomplete আসবে)
Role: Core Router
Location: Mirpur POP (Hierarchy দেখাবে: Dhaka North Zone > Mirpur Cluster > Mirpur POP)
```

এই চারটা ফিল্ড সবচেয়ে ইম্পর্ট্যান্ট। এগুলো required।

এখন optional কিন্তু recommended ফিল্ডগুলো:

```
Rack: Rack A
Position: 25 (মানে র‍্যাকের ২৫U পজিশনে)
Face: Front

Status: Active

Tenant: (খালি রাখুন - Nirvor Communication একটাই organization)
Platform: (খালি রাখুন - এটা OS platform এর জন্য, optional)

Serial Number: ABC1234MIR001
Asset Tag: SKY-RTR-001

Description: Primary core router for Mirpur POP serving approximately 4000 customers

Comments:
  - Purchased: December 2022
  - Vendor: TechSource Bangladesh
  - Purchase Order: PO-2022-12-015
  - Warranty: 3 years (expires Dec 2025)
  - Installation Date: January 15, 2023
```

সব পূরণ হয়ে গেলে **Create** ক্লিক করুন।

আপনার প্রথম ডিভাইস তৈরি হয়ে গেছে!

ডিভাইস ডিটেইল পেজে চলে যাবেন। এখানে দেখবেন:

- **Device Info:** নাম, টাইপ, লোকেশন, status
- **Rack Position:** একটা visual diagram যেখানে র‍্যাকে ডিভাইসের পজিশন দেখাচ্ছে
- **Interfaces:** সব ইন্টারফেসের লিস্ট (১৫টা - Device Type থেকে কপি হয়েছে)

**Interfaces** ট্যাবে ক্লিক করুন। দেখবেন:

```
ether1
sfp-sfpplus1
sfp-sfpplus2
...
sfp-sfpplus12
sfp28-1
sfp28-2
```

চমৎকার! সব ইন্টারফেস অটোমেটিক্যালি এসেছে।

---

### Interface কনফিগারেশন - Details যোগ করা

এখন ইন্টারফেসগুলোতে কিছু ডিটেইলস যোগ করব।

ether1 ইন্টারফেসে ক্লিক করুন। একটা ডিটেইল পেজ আসবে। **Edit** বাটনে ক্লিক করুন।

```
Name: ether1 (ইতিমধ্যে আছে)
Type: 1000BASE-T (ইতিমধ্যে সেট)

Label: MGMT (optional - physical label)
Enabled: ✓ (চেক করুন - এই পোর্ট চালু আছে)

Speed: 1000000 (1 Gbps - Kbps এ)
Duplex: Full

MTU: 1500

Mode: Access (না Tagged)

Description: Management interface - connected to management VLAN
```

**Update** ক্লিক করুন।

একইভাবে uplink interface:

sfp-sfpplus1 এ যান → **Edit** ক্লিক করুন:

```
Label: UPLINK-BTCL
Enabled: ✓
Speed: 10000000 (10 Gbps)
MTU: 1500
Description: Primary uplink to BTCL - 5 Gbps circuit
```

**Update** করুন।

আপনি চাইলে সব ইন্টারফেসে description দিতে পারেন। তবে শুধু যেগুলো ব্যবহার হচ্ছে সেগুলোতে দিলেই চলবে।

---

### আরো Devices যোগ করা - Bulk এর জন্য প্রস্তুতি

এখন আরো কিছু ডিভাইস যোগ করব। Distribution switch এবং access switches।

**Distribution Switch - মিরপুর:**

**Devices → Devices → + Add**

```
Name: SW-DN-MIR-DIST-01
Device Type: TP-Link TL-SG3428
Role: Distribution Switch
Location: Mirpur POP
Rack: Rack A
Position: 20
Face: Front
Status: Active
Serial Number: TPL1234DIST01
Asset Tag: SKY-SW-001
Description: Primary distribution switch for Mirpur POP
```

**Create** করুন।

**Access Switches - একটা একটা করে (পরে CSV দেখাব):**

প্রথম Access Switch:

```
Name: SW-DN-MIR-ACC-01
Device Type: TP-Link TL-SG1024D
Role: Access Switch
Location: Mirpur POP
Rack: Rack B
Position: 15
Status: Active
Serial Number: TPLACC001
Asset Tag: SKY-SW-011
Description: Building A access switch
```

দ্বিতীয় Access Switch:

```
Name: SW-DN-MIR-ACC-02
Device Type: TP-Link TL-SG1024D
Role: Access Switch
Location: Mirpur POP
Rack: Rack B
Position: 14
Status: Active
Serial Number: TPLACC002
Asset Tag: SKY-SW-012
Description: Building B access switch
```

এভাবে একটা একটা করে করা বিরক্তিকর। দশটা access switch থাকলে তো পুরো দিন চলে যাবে!

---

### CSV Bulk Import - দ্রুত ডেটা এন্ট্রি

Nautobot এ CSV দিয়ে bulk import করার সুবিধা আছে। চলুন দেখি কীভাবে।

#### CSV Template ডাউনলোড করা

**Devices → Devices** লিস্ট পেজে যান।

উপরে ডান দিকে **Import** বাটন দেখবেন। ক্লিক করুন।

একটা পেজ আসবে যেখানে CSV ফরম্যাট দেখানো আছে। নিচে **CSV Field Mapping** সেকশনে সব ফিল্ডের লিস্ট আছে।

#### CSV ফাইল তৈরি করা

Excel বা Google Sheets এ একটা নতুন ফাইল তৈরি করুন।

প্রথম রো তে হেডার:

```csv
name,device_type,role,location,rack,position,status,serial,asset_tag,description
```

এখন ডেটা রো:

```csv
name,device_type,role,location,rack,position,status,serial,asset_tag,description
SW-DN-MIR-ACC-03,TP-Link TL-SG1024D,Access Switch,Mirpur POP,Rack B,13,Active,TPLACC003,SKY-SW-013,Building C access switch
SW-DN-MIR-ACC-04,TP-Link TL-SG1024D,Access Switch,Mirpur POP,Rack B,12,Active,TPLACC004,SKY-SW-014,Building D access switch
SW-DN-MIR-ACC-05,TP-Link TL-SG1024D,Access Switch,Mirpur POP,Rack B,11,Active,TPLACC005,SKY-SW-015,Building E access switch
```

**গুরুত্বপূর্ণ নোট:**

- Device Type, Role, Location, Rack এর নাম হুবহু Nautobot এ যেভাবে লেখা আছে সেভাবে লিখতে হবে
- Status ও exact মিলতে হবে (Active, Planned, Offline ইত্যাদি)
- Position একটা নাম্বার
- Serial এবং Asset Tag unique হতে হবে

ফাইল CSV ফরম্যাটে সেভ করুন (`devices.csv`)।

#### CSV ইমপোর্ট করা

**Devices → Devices → Import** যান।

**CSV Data** বক্সে আপনার CSV ফাইলের কন্টেন্ট পেস্ট করুন। অথবা **Browse** বাটনে ক্লিক করে ফাইল আপলোড করুন।

**Submit** ক্লিক করুন।

Nautobot CSV parse করবে এবং একটা preview দেখাবে:

```
Found 3 objects to import:
  - SW-DN-MIR-ACC-03
  - SW-DN-MIR-ACC-04
  - SW-DN-MIR-ACC-05
```

যদি কোনো এরর থাকে (যেমন Device Type নাম ভুল), তাহলে সেটা লাল করে দেখাবে।

সব ঠিক থাকলে **Confirm Import** বাটনে ক্লিক করুন।

তিনটা ডিভাইস একসাথে তৈরি হয়ে যাবে!

**Devices → Devices** লিস্টে গিয়ে দেখুন - এখন আপনার মোট ৭টা ডিভাইস আছে:
- ১টা Router
- ১টা Distribution Switch
- ৫টা Access Switch

---

### IP Address Assignment - Logical কানেকশন

এখন ডিভাইসগুলোতে IP address assign করব।

#### প্রথমে IP Namespace তৈরি

**IPAM → Namespaces** যান। **+ Add** ক্লিক করুন।

```
Name: Global
Description: Primary IP namespace for Nirvor Communication Bangladesh
```

**Create** করুন।

#### IP Prefixes তৈরি করা

IP addresses assign করার আগে prefixes তৈরি করতে হবে।

**IPAM → Prefixes** যান। **+ Add** ক্লিক করুন।

**Parent Prefix - Private Management Network:**

```
Prefix: 10.10.0.0/16
Type: Container
Status: Active
Namespace: Global
Description: Internal management network for all Nirvor Communication equipment
```

**Create and Add Another** করুন।

**Child Prefix - Mirpur Management:**

```
Prefix: 10.10.10.0/24
Type: Network
Status: Active
Namespace: Global
Parent: 10.10.0.0/16 (ড্রপডাউন থেকে সিলেক্ট করুন)
Location: Mirpur POP
Description: Management IPs for Mirpur POP devices
```

**Create** করুন।

#### IP Addresses তৈরি এবং Interface এ Assign করা

**IPAM → IP Addresses** যান। **+ Add** ক্লিক করুন।

**Core Router Loopback:**

```
IP Address: 10.10.1.1/32
Type: Host
Status: Active
Role: Loopback (ড্রপডাউন থেকে)
Namespace: Global
Parent Prefix: (খালি রাখুন - auto-detect হবে)

Assigned Object Type: dcim > interface
Assigned Object: 
  - Device: R-DN-MIR-CORE-01
  - Interface: ether1 (ড্রপডাউনে দেখাবে)

DNS Name: r-mir-core-01.nirvor.bd
Description: Management loopback for Mirpur core router
```

**Create** করুন।

লক্ষ্য করুন - Assigned Object টা একটু tricky। প্রথমে Type সিলেক্ট করতে হবে `dcim > interface`, তারপর দুটো ড্রপডাউন আসবে - Device এবং Interface।

**Distribution Switch Management IP:**

```
IP Address: 10.10.10.11/24
Type: Host
Status: Active
Namespace: Global

Assigned Object Type: dcim > interface
Assigned Object:
  - Device: SW-DN-MIR-DIST-01
  - Interface: (এখানে একটু সমস্যা - TP-Link switch এ VLAN interface নেই)
```

একটু দাঁড়ান - TP-Link TL-SG3428 এ management VLAN interface যোগ করা হয়নি। চলুন যোগ করি।

**Devices → Devices → SW-DN-MIR-DIST-01** এ যান।

**Interfaces** ট্যাবে → **+ Add Interfaces** ক্লিক করুন।

```
Name: vlan10
Type: Virtual
Enabled: ✓
Description: Management VLAN interface
```

**Create** করুন।

এখন আবার IP address assignment এ ফিরে যান:

```
IP Address: 10.10.10.11/24
Assigned Object:
  - Device: SW-DN-MIR-DIST-01
  - Interface: vlan10

DNS Name: sw-mir-dist-01.nirvor.bd
```

**Create** করুন।

একইভাবে access switches এ:

```
10.10.10.21/24 → SW-DN-MIR-ACC-01 (vlan10 interface তৈরি করে)
10.10.10.22/24 → SW-DN-MIR-ACC-02
10.10.10.23/24 → SW-DN-MIR-ACC-03
...
```

এগুলো CSV দিয়েও করা যায়, কিন্তু Interface assignment একটু tricky CSV এ।

#### VLAN Management - Layer 2 সেগমেন্টেশন

Nirvor Communication এর নেটওয়ার্কে বিভিন্ন ধরনের ট্রাফিক আছে - ম্যানেজমেন্ট, কাস্টমার, আপলিংক। এগুলো আলাদা করার জন্য VLAN ব্যবহার করা হয়।

##### VLAN Groups তৈরি করা (অপশনাল কিন্তু recommended)

VLAN Groups দিয়ে VLANs organize করা যায়। বিশেষ করে যখন একই VLAN ID বিভিন্ন সাইটে ভিন্ন purpose এ ব্যবহার হয়।

**IPAM → VLAN Groups** যান। **+ Add** ক্লিক করুন।

```
Name: Mirpur POP VLANs
Location: Mirpur POP
Description: VLAN assignments for Mirpur Point of Presence
```

**Create** করুন।

একইভাবে Uttara POP এর জন্য:

```
Name: Uttara POP VLANs
Location: Uttara POP
Description: VLAN assignments for Uttara Point of Presence
```

#### VLANs তৈরি করা

এখন actual VLANs তৈরি করব।

**IPAM → VLANs** যান। **+ Add** ক্লিক করুন।

**VLAN 10 - Management (Mirpur):**

```
VLAN ID: 10
Name: MGMT_MIRPUR
Status: Active
Group: Mirpur POP VLANs (ড্রপডাউন থেকে)
Location: Mirpur POP
Tenant: (খালি)
Description: Management VLAN for network devices in Mirpur POP
```

**Create and Add Another** করুন।

**VLAN 20 - Uplink (Mirpur):**

```
VLAN ID: 20
Name: UPLINK_MIRPUR
Status: Active
Group: Mirpur POP VLANs
Location: Mirpur POP
Description: Uplink traffic to BTCL from Mirpur
```

**Create and Add Another** করুন।

**VLAN 100 - Residential (Mirpur):**

```
VLAN ID: 100
Name: RESIDENTIAL_MIR
Status: Active
Group: Mirpur POP VLANs
Location: Mirpur POP
Description: Residential customer traffic - Mirpur area
```

**Create** করুন।

একইভাবে Uttara POP এর জন্য VLANs তৈরি করুন:

```
VLAN 10: MGMT_UTTARA (Group: Uttara POP VLANs, Location: Uttara POP)
VLAN 21: UPLINK_UTTARA (Group: Uttara POP VLANs)
VLAN 101: RESIDENTIAL_UTT (Group: Uttara POP VLANs)
```

**Site-Agnostic VLAN (সব সাইটে একই):**

কিছু VLAN সব সাইটে একই purpose এ ব্যবহার হয়:

```
VLAN ID: 200
Name: CORPORATE
Status: Active
Group: (খালি - কোনো specific group এ নেই)
Location: (খালি - সব সাইটে ব্যবহার হয়)
Description: Corporate client connections across all POPs
```

এখন **IPAM → VLANs** লিস্টে গিয়ে দেখুন - আপনার সব VLANs দেখাবে VLAN ID, Name, Location সহ।

#### VLAN এবং Prefix এর সম্পর্ক

এখন VLANs এর সাথে IP Prefixes link করব।

**IPAM → Prefixes** এ যান। আগে তৈরি করা 10.10.10.0/24 prefix খুঁজুন। ক্লিক করে ডিটেইল পেজে যান।

**Edit** বাটনে ক্লিক করুন।

```
Prefix: 10.10.10.0/24
...
(সব ফিল্ড আগের মতো)

VLAN: MGMT_MIRPUR (এটা নতুন যোগ করুন - ড্রপডাউন থেকে সিলেক্ট)
```

**Update** করুন।

এখন Nautobot জানে 10.10.10.0/24 prefix টা VLAN 10 (MGMT_MIRPUR) এ ব্যবহার হয়।

---

### Cable Management - Physical Connections

এখন ডিভাইসগুলোর মধ্যে physical cable connections ডকুমেন্ট করব।

#### Cable তৈরি করা - Method 1 (Interface থেকে)

সবচেয়ে সহজ পদ্ধতি হলো একটা interface থেকে সরাসরি cable connect করা।

**Devices → Devices → R-DN-MIR-CORE-01** এ যান।

**Interfaces** ট্যাবে ক্লিক করুন।

**sfp-sfpplus2** interface খুঁজুন (এটা distribution switch এ যাবে)। ক্লিক করুন।

Interface ডিটেইল পেজে **Connect Cable** বাটন দেখবেন। ক্লিক করুন।

একটা ফর্ম আসবে:

```
Termination A (already filled):
  - Device: R-DN-MIR-CORE-01
  - Interface: sfp-sfpplus2

Termination B:
  - Termination Type: Interface (default selected)
  - Device: SW-DN-MIR-DIST-01 (ড্রপডাউন থেকে সিলেক্ট করুন)
  - Name: sfp1 (এই switch এর SFP port)

Cable:
  - Type: Single-Mode Fiber (SMF) (ড্রপডাউন থেকে)
  - Status: Connected
  - Color: 0000ff (Blue - হেক্স কোড)
  - Length: 5
  - Length Unit: Meters
  - Label: CORE-TO-DIST-01

Description: Core router to distribution switch - 10G fiber link
```

**Create** ক্লিক করুন।

Cable তৈরি হয়ে গেছে! এখন R-DN-MIR-CORE-01 এর sfp-sfpplus2 interface এ গেলে দেখবেন "Cable: CORE-TO-DIST-01" লেখা আছে। ক্লিক করলে cable details দেখাবে।

#### Cable তৈরি করা - Method 2 (সরাসরি Cable object)

আরেকটা পদ্ধতি হলো সরাসরি Cable object তৈরি করা।

**Devices → Cables** যান। **+ Add** ক্লিক করুন।

```
Termination A:
  - Termination Type: dcim > interface
  - Device: SW-DN-MIR-DIST-01
  - Name: ether1

Termination B:
  - Termination Type: dcim > interface
  - Device: SW-DN-MIR-ACC-01
  - Name: ether24

Cable:
  - Type: Cat6 (Category 6 Ethernet)
  - Status: Connected
  - Color: ffeb3b (Yellow)
  - Length: 20
  - Length Unit: Meters
  - Label: DIST-TO-ACC-01

Description: Distribution to access switch - Building A
```

**Create** করুন।

একইভাবে আরো কয়েকটা cable তৈরি করুন:

**Distribution to Access Switch 02:**

```
SW-DN-MIR-DIST-01 (ether2) ↔ SW-DN-MIR-ACC-02 (ether24)
Type: Cat6, Color: Yellow, Length: 22m
Label: DIST-TO-ACC-02
```

**Distribution to Access Switch 03:**

```
SW-DN-MIR-DIST-01 (ether3) ↔ SW-DN-MIR-ACC-03 (ether24)
Type: Cat6, Color: Yellow, Length: 25m
Label: DIST-TO-ACC-03
```

#### Cable Types - কোনগুলো available?

Nautobot এ built-in cable types আছে:

**Copper:**
- Cat3, Cat5, Cat5e, Cat6, Cat6a, Cat7, Cat8
- Coaxial
- Direct Attach Copper (DAC)

**Fiber:**
- Multi-Mode Fiber (MMF)
- Single-Mode Fiber (SMF)
- Active Optical Cabling (AOC)

**Power:**
- AC Power
- DC Power

আপনার যেটা দরকার সেটা সিলেক্ট করুন।

#### Cable Trace ব্যবহার করা

এখন একটা powerful ফিচার দেখি - Cable Trace।

**Devices → Devices → SW-DN-MIR-ACC-01** এ যান।

**Interfaces** ট্যাবে → **ether24** interface ক্লিক করুন।

Interface পেজে **Trace** বাটন দেখবেন। ক্লিক করুন।

একটা visual diagram দেখাবে:

```
SW-DN-MIR-ACC-01 (ether24)
    |
    | Cable: DIST-TO-ACC-01 (Cat6, 20m, Yellow)
    ↓
SW-DN-MIR-DIST-01 (ether1)
    |
    | Cable: CORE-TO-DIST-01 (SMF, 5m, Blue)
    ↓
R-DN-MIR-CORE-01 (sfp-sfpplus2)
```

এভাবে পুরো physical path দেখতে পাবেন। এটা troubleshooting এর সময় অসাধারণ কাজে লাগে।

---

### Provider এবং Circuit সেটআপ

Nirvor Communication এর upstream connectivity BTCL থেকে। চলুন এটা ডকুমেন্ট করি।

#### Provider তৈরি করা

**Circuits → Providers** যান। **+ Add** ক্লিক করুন।

```
Name: BTCL
ASN: 17494
Account Number: SKYN-BTCL-2023-MIR
Portal URL: https://isp.btcl.gov.bd
NOC Contact: +880 2-9555555
Admin Contact: noc@btcl.gov.bd

Description: Bangladesh Telecommunications Company Limited - Primary upstream provider

Comments:
  - Account Manager: Mr. Kamal Hossain
  - Sales Contact: +880 2-9555556
  - Technical Support: support@btcl.gov.bd
  - 24/7 NOC: +880 1700-000000
```

**Create** করুন।

#### Circuit Type তৈরি করা

**Circuits → Circuit Types** যান। **+ Add** ক্লিক করুন।

```
Name: Fiber Optic
Description: Fiber optic connectivity - single-mode or multi-mode
```

**Create** করুন।

#### Circuit তৈরি করা

**Circuits → Circuits** যান। **+ Add** ক্লিক করুন।

```
Circuit ID: BTCL-MIR-001
Provider: BTCL (ড্রপডাউন থেকে)
Type: Fiber Optic
Status: Active

Install Date: 2023-03-15
Termination Date: (খালি - চলমান)
Commit Rate: 5000 (Mbps - ৫ Gbps)

Description: Primary 5Gbps internet uplink from Mirpur POP

Comments:
  - Contract Start: 2023-03-01
  - Contract Duration: 2 years
  - Monthly Cost: BDT 400,000
  - Contract Renewal Date: 2025-03-01
  - Bandwidth Burstable: Up to 6 Gbps
  - SLA: 99.5% uptime guarantee
```

**Create** করুন।

এখন Circuit Terminations যোগ করতে হবে।

Circuit ডিটেইল পেজে **Terminations** ট্যাবে যান। **+ Add Termination** ক্লিক করুন।

**Termination A (Provider Side):**

```
Circuit: BTCL-MIR-001 (already selected)
Term Side: A
Location: (খালি - provider সাইড)
Provider Network: (খালি)
Port Speed: 5000 (Mbps)
Upstream Speed: 5000 (Mbps)
Cross Connect ID: BTCL-XC-MIR-12345
Patch Panel/Port: PP-BTCL-MIR-001
Description: BTCL side termination at their Mirpur exchange
```

**Create and Add Another** করুন।

**Termination Z (Nirvor Communication Side):**

```
Circuit: BTCL-MIR-001
Term Side: Z
Location: Mirpur POP (ড্রপডাউন থেকে)
Device: R-DN-MIR-CORE-01
Interface: sfp-sfpplus1
Port Speed: 5000 (Mbps)
Description: Connected to core router uplink port
```

**Create** করুন।

এখন Circuit এবং Device পুরোপুরি linked। Circuit ডিটেইল পেজে দেখবেন Termination Z এ device এবং interface এর লিংক আছে। ক্লিক করলে সরাসরি সেই interface এ যাবেন।

একইভাবে উত্তরা পপের জন্য Summit Communications এর circuit তৈরি করুন (যদি থাকে)।

---

### Data Validation এবং Quality Check

এতক্ষণ অনেক ডেটা এন্ট্রি করলাম। এখন চেক করা দরকার সব ঠিক আছে কিনা।

#### Device List Review

**Devices → Devices** যান।

উপরে ডান দিকে **Filters** প্যানেল দেখবেন। এখান থেকে filter করতে পারবেন:

**Location দিয়ে filter:**
- Location: Mirpur POP সিলেক্ট করুন
- শুধু মিরপুর পপের ডিভাইস দেখাবে

**Role দিয়ে filter:**
- Role: Core Router সিলেক্ট করুন
- শুধু core routers দেখাবে

**Status দিয়ে filter:**
- Status: Active সিলেক্ট করুন
- শুধু active ডিভাইস দেখাবে

#### Missing Information চেক করা

কিছু common issues খুঁজুন:

**Devices without Serial Numbers:**

Filters প্যানেলে:
- Has Serial Number: No সিলেক্ট করুন

যদি কোনো device আসে, তাহলে সেগুলোতে serial number যোগ করুন।

**Devices without Location:**

Filters:
- Location: (empty) - অর্থাৎ কোনো location নেই

এরকম device থাকা উচিত না। থাকলে সংশোধন করুন।

#### IP Address Utilization দেখা

**IPAM → Prefixes** যান।

10.10.10.0/24 prefix ক্লিক করুন।

ডিটেইল পেজে দেখবেন:

```
Utilization: 23% (6 of 256 IPs used)
```

একটা ভিজুয়াল বার দেখাবে কতটুকু ব্যবহৃত। যদি 80% এর উপরে যায়, তাহলে নতুন prefix প্ল্যান করুন।

**IP Addresses** ট্যাবে ক্লিক করলে এই prefix এর সব IPs দেখাবে:

```
10.10.10.11/24 - SW-DN-MIR-DIST-01 (Active)
10.10.10.21/24 - SW-DN-MIR-ACC-01 (Active)
10.10.10.22/24 - SW-DN-MIR-ACC-02 (Active)
...
```

#### Duplicate Name চেক

Nautobot automatically duplicate names prevent করে, কিন্তু double-check করা ভালো।

**Devices → Devices** লিস্টে name column এ sort করুন (column header এ ক্লিক করুন)। দেখুন কোনো duplicate আছে কিনা।

---

### Common Mistakes এবং কীভাবে এড়াবেন

Nirvor Communication এর NOC টিম প্রথমে কিছু ভুল করেছিল। চলুন সেগুলো দেখি যাতে আপনি এড়াতে পারেন।

#### Mistake 1: Location Hierarchy ভুল

**ভুল করা হয়েছিল:**

Mirpur POP তৈরি করার সময় Parent Location দেওয়া হয়নি। ফলে এটা top-level এ চলে গিয়েছিল।

```
(ভুল hierarchy)
Dhaka North Zone
Mirpur Cluster
Mirpur POP (parent নেই - top level এ!)
```

**সমাধান:**

Mirpur POP edit করে Parent Location = Mirpur Cluster সেট করতে হয়েছে।

**শিক্ষা:** Location তৈরি করার সময় সবসময় Parent Location চেক করুন।

#### Mistake 2: Device Type এ Interface Templates ভুলে যাওয়া

**ভুল:**

TP-Link TL-SG1024D device type তৈরি করা হয়েছিল কিন্তু interface templates যোগ করা হয়নি। ফলে device তৈরি করার পরে manually ২৪টা interface যোগ করতে হয়েছে প্রতিটা device এ!

**সমাধান:**

Device Type এ একবার interface templates ঠিকমতো যোগ করার পরে সব ঠিক হয়েছে।

**শিক্ষা:** Device Type তৈরি করার সময় অবশ্যই Interface Templates যোগ করুন। একটু সময় লাগবে, কিন্তু পরে অনেক সময় বাঁচবে।

#### Mistake 3: IP Address এ Prefix Mask ভুল

**ভুল:**

Management IP দেওয়া হয়েছিল 10.10.10.11/32 (একটা host IP হিসেবে)। কিন্তু আসলে এটা একটা network এর part, তাই হওয়া উচিত ছিল 10.10.10.11/24।

**সমস্যা:**

/32 দিলে ওই IP শুধু নিজের সাথে communicate করতে পারে, network এর অন্য কারো সাথে না!

**সমাধান:**

সব IP addresses review করে সঠিক subnet mask দিতে হয়েছে।

**শিক্ষা:** 
- Host IPs (loopbacks, point-to-point links): /32 বা /30, /31
- Network IPs (management, LANs): /24, /25, /26 ইত্যাদি

#### Mistake 4: Cable Color Coding না মানা

**ভুল:**

প্রথমে সব cable এ random color দেওয়া হয়েছিল। ফলে physical rack এ গিয়ে কোন cable কী বোঝা যাচ্ছিল না।

**সমাধান:**

একটা color coding scheme বানানো হয়েছে:
- Blue: Fiber cables
- Yellow: Cat6 ethernet (access to distribution)
- Green: Cat6 ethernet (customer connections)
- Red: Critical uplink/backbone

Nautobot এ সব cable update করা হয়েছে এই color scheme অনুযায়ী।

**শিক্ষা:** একটা consistent naming এবং color scheme follow করুন।

#### Mistake 5: Description ফিল্ড খালি রাখা

**ভুল:**

প্রথম দিকে শুধু device name দেওয়া হয়েছিল, description খালি ছিল।

```
Name: SW-DN-MIR-ACC-03
Description: (empty)
```

ছয় মাস পরে যখন দেখা গেল, কেউ বুঝতে পারছিল না এই switch কোন building এ, কী purpose এ।

**সমাধান:**

সব device এ proper description যোগ করা হয়েছে:

```
Name: SW-DN-MIR-ACC-03
Description: Building C access switch serving 280 residential customers in Mirpur Sector 12
```

**শিক্ষা:** Description এবং Comments ফিল্ড সবসময় পূরণ করুন। ভবিষ্যতে নিজেই নিজেকে ধন্যবাদ দেবেন।

#### Mistake 6: Status সঠিকভাবে সেট না করা

**ভুল:**

একটা old switch যেটা আর ব্যবহার হয় না, সেটার status ছিল "Active"।

**সমস্যা:**

Reports এ এটা count হচ্ছিল। ফলে actual active devices count ভুল আসছিল।

**সমাধান:**

Device status "Decommissioned" করা হয়েছে। অথবা একদম delete করে দেওয়া হয়েছে।

**শিক্ষা:** Device lifecycle maintain করুন - Active, Offline, Decommissioned, Retired ইত্যাদি সঠিক status দিন।

---

### Naming Convention Standards - Nirvor Communication এর Example

Nirvor Communication একটা documented naming convention তৈরি করেছে। এটা একটা Word ডকুমেন্টে রাখা আছে এবং সব team members কে দেওয়া হয়েছে।

```
========================================
Nirvor Communication Naming Convention
Version 1.0 - Updated: February 2025
========================================

1. DEVICES
----------
Format: [Type]-[Zone]-[Site]-[Role]-[Number]

Type Codes:
  R  = Router
  SW = Switch
  FW = Firewall
  OLT = Optical Line Terminal

Zone Codes:
  DN = Dhaka North
  DS = Dhaka South (future)
  CT = Chittagong (future)

Site Codes (3 letters):
  MIR = Mirpur
  UTT = Uttara
  KAL = Kalyanpur (future)
  BAN = Banani (future)

Role Codes:
  CORE = Core
  DIST = Distribution
  ACC  = Access
  EDGE = Edge

Number: 01, 02, 03... (zero-padded, 2 digits)

Examples:
  R-DN-MIR-CORE-01
  SW-DN-UTT-DIST-01
  SW-DN-MIR-ACC-15

2. INTERFACES
-------------
Use manufacturer's default naming.
Add clear descriptions:

Format: [Purpose] - [Remote End/Details]

Examples:
  "Uplink to BTCL - 5 Gbps circuit"
  "Link to SW-DN-MIR-DIST-01 port sfp1"
  "Customer: ABC Corporation - VLAN 200"

3. CABLES
---------
Format: [Source Device Type]-TO-[Dest Device Type]-[Number]

Examples:
  CORE-TO-DIST-01
  DIST-TO-ACC-03
  UPLINK-BTCL-01

4. IP ADDRESSES
---------------
Management IPs:
  Core Routers: 10.10.X.1/24
  Dist Switches: 10.10.X.11-20/24
  Access Switches: 10.10.X.21-99/24

DNS Naming:
  Format: [device-name].nirvor.bd
  Example: r-mir-core-01.nirvor.bd

5. VLANs
--------
Format: [Purpose]_[Site]

Examples:
  MGMT_MIRPUR
  RESIDENTIAL_MIR
  CORPORATE (global, no site suffix)

6. LOCATIONS
------------
Use full descriptive names:
  Dhaka North Zone
  Mirpur POP
  Rack A

Avoid abbreviations in location names.

========================================
```

এই document টা print করে NOC তে টাঙিয়ে রাখা আছে। নতুন কেউ join করলে তাকে এটা পড়তে দেওয়া হয়।

---

### Export এবং Backup - ডেটা সংরক্ষণ

কিছু ডেটা এন্ট্রি হয়ে গেলে backup নেওয়া জরুরি।

#### CSV Export

যেকোনো object list থেকে CSV export করা যায়।

**Devices → Devices** লিস্টে যান।

উপরে ডান দিকে **Export** বাটন দেখবেন। ক্লিক করলে একটা ড্রপডাউন:

- **Export All**: সব devices
- **Export Selected**: যেগুলো checkbox দিয়ে সিলেক্ট করেছেন
- **Export as CSV**: CSV format
- **Export as YAML**: YAML format

**Export All → CSV** সিলেক্ট করুন।

একটা CSV ফাইল ডাউনলোড হবে যেখানে সব devices এর সব information থাকবে।

এই ফাইল একটা safe জায়গায় রাখুন। যদি কখনো ভুল করে কিছু delete হয়ে যায়, এই CSV দিয়ে import করে recover করতে পারবেন।

#### Monthly Data Snapshot

Nirvor Communication প্রতি মাসের ১ তারিখে একটা snapshot নেয়:

```bash
# Create backup directory
mkdir -p /backup/nautobot/monthly/2025-02

# Export all key data types
# Devices
curl -H "Authorization: Token YOUR_TOKEN" \
  "https://nautobot.nirvor.bd/api/dcim/devices/?limit=0&format=csv" \
  > /backup/nautobot/monthly/2025-02/devices.csv

# IP Addresses
curl -H "Authorization: Token YOUR_TOKEN" \
  "https://nautobot.nirvor.bd/api/ipam/ip-addresses/?limit=0&format=csv" \
  > /backup/nautobot/monthly/2025-02/ip-addresses.csv

# (similarly for other object types)
```

এগুলো Google Drive এ backup করা হয়।

---

### চ্যাপ্টার সারাংশ

এই চ্যাপ্টারে আমরা শিখলাম:

**Location Hierarchy:**
- Location Types তৈরি করা (Zone, Cluster, POP, Rack)
- Actual Locations তৈরি করা
- Hierarchy সঠিকভাবে সেটআপ করা

**Roles Setup:**
- Unified Roles সিস্টেম (Nautobot 3.0)
- Content Types দিয়ে specify করা
- Device roles তৈরি করা

**Device Management:**
- Manufacturers যোগ করা
- Device Types তৈরি করা
- Interface Templates এর গুরুত্ব
- Actual Devices এন্ট্রি করা
- Racks এ device position করা

**IP এবং VLAN:**
- IP Namespaces
- Prefixes (hierarchical)
- IP Addresses assign করা
- VLANs তৈরি করা
- VLAN Groups

**Physical Connectivity:**
- Cables তৈরি করা
- Cable Trace ব্যবহার করা
- Color coding

**Upstream Connectivity:**
- Providers
- Circuit Types
- Circuits
- Circuit Terminations

**Data Quality:**
- Validation checks
- Common mistakes এবং সমাধান
- Naming conventions
- Export/Backup

Nirvor Communication এখন তাদের মিরপুর এবং উত্তরা পপের প্রায় সম্পূর্ণ নেটওয়ার্ক Nautobot এ ডকুমেন্ট করে ফেলেছে। পরের চ্যাপ্টারে আমরা দেখব কীভাবে এই setup কে scale করা যায় যখন নতুন পপ যুক্ত হয়, আরো ডিভাইস আসে, এবং কীভাবে advanced features ব্যবহার করতে হয়।
