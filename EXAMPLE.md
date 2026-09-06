# Cellula Ontologies — Illustrative Example

This file continues [README.md](README.md), which describes the Category, Cell, Graph, Persona, and Organization ontologies, and is continued by [APP-BEHAVIOR.md](APP-BEHAVIOR.md), which documents how the app behaves on top of this data. It provides an illustrative example — a hypothetical user, Alice Walker — showing how those ontologies are used together, followed by diagram-generation instructions and the full validation pipeline for the example dataset.

## Illustrative Example: Alice

This section describes the local dataset for a hypothetical user, Alice Walker. Alice's cells — each a folder holding exactly one cell DataBook file — live in a tree of cells rooted at `example/Cells/`. Every mention of "Self" in the following is a reference to the user, Alice.

### Bob and Fred

Alice knows two people, Bob and Fred. Under *Others* she has created a two-member cell for each, sharing one with Bob and the other with Fred.

In her shared cell with Bob ([cell 16](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md>)) Alice has included some claims about herself ([graph 12](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-12>)) including her given name "Alice", her family name "Walker", etc. She has included ([graph 4](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-04>)) her claim that Bob's favorite drink is an oat milk cappuccino. Bob has claimed some contact information about himself ([graph 2](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-02>)), and he claims that her favorite drink is Pepsi ([graph 8](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-08>)).

<p align="center"><img src="example/images/people.png" alt="People cells"></p>

### Taking Care of Sophia

To capture Alice's family-related relationship with her daughter, Sophia Walker, Alice created a cell ([cell 12](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md>)) named *Sophia Walker*, nested under her *Immediate Family* cell. Its two members are Alice herself ([graph 21](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md#graph-21>)) and her husband Dave ([graph 5](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md#graph-05>)). Sophia has no instance of the app, so she cannot join the cell as a member: Alice added a graph about her by hand as the cell's `c:topic` and chose the *Contact Info* template (`pshapes:ContactInfoShape`) for it ([graph 7](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md#graph-07>)). That manual step is what makes an ordinary `cat:ImmediateFamily` cell — a category whose template cell carries `c:isTopicCell false` — a `c:TopicCell` all the same, with Sophia as its derived subject (see integrity.md's Check 31).

Alice spends time taking care of her daughter, so she has assembled some information about Sophia herself, in cells she has not shared. In the *Health & Wellness* cell ([cell 13](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md>)) Alice keeps a record of Sophia's physical characteristics such as height, eye color, and hair color in [graph 17](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md#graph-17>). This is a single-member cell whose subject is Sophia. Its required `member` slot holds a minimal graph about Alice herself ([graph 35](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md#graph-35>)). [Graph 17](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md#graph-17>) is linked via `c:topic`.

Under *Medical* > *Provider* > *Primary Care Physician*, Alice keeps a record of Dr. Jane Starostina, Sophia's primary care physician ([graph 25](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Jane Starostina/Jane Starostina(primary-care-physician).databook.md#graph-25>)). This is a single-member cell whose subject is Jane.

Alice's husband Dave is involved in taking care of their daughter. The two parents need to arrange medical appointments, etc. To do so, they need to share and synchronize medical information about Sophia, including her list of medications, medical history, health insurance policy, contact information and so on. To work on this as a team, Alice creates a two-member *Medical Appointment* cell and shares it with Dave. They both use it to share information about Sophia's upcoming medical appointment ([graph 26](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Medical Appointment/Medical Appointment.databook.md#graph-26>)). This graph includes the name of Sophia's doctor (primary care physician) which the app copies from the Dr. Jane Starostina cell ([graph 25](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Jane Starostina/Jane Starostina(primary-care-physician).databook.md#graph-25>)).

<p align="center"><img src="example/images/people2.png" alt="People cells, continued — Immediate Family, Sophia Walker, and her Health & Wellness, Medical, and Provider cells"></p>

### Working for Acme

Alice is an employee of Acme, so under her *Work* cell she has created an *Acme* cell to represent her employer. Since Acme is an organization, rather than using `cat:Person` categories she has switched to `cat:Organization` categories (light blue color).

Under *Employees* she has added her own *Alice Walker* cell holding her Business Card claims ([graph 10](<example/Cells/Work/Acme/Employees/Alice Walker/Alice Walker(employees).databook.md#graph-10>)) — job title at Acme, work telephone number, work email, etc. One of the employees she works with is Paula Walker, so she has a *Paula Walker* cell for her — a two-member cell with Paula herself as the second member ([graph 6](<example/Cells/Work/Acme/Employees/Paula Walker/Paula Walker(employees).databook.md#graph-06>), a bare identifying claim mirroring Alice's own) alongside Alice's own claims about herself ([graph 20](<example/Cells/Work/Acme/Employees/Paula Walker/Paula Walker(employees).databook.md#graph-20>)) — neither of which has been shared with Paula, since Alice has not yet shared the cell with her: both graphs are still her own claims, graph 6 included.

<p align="center"><img src="example/images/work.png" alt="Work cells"></p>

### Service Providers

Alice has relationships with two companies, Google and AT&T (her cell phone provider). Both are `cat:Companies` `c:TopicCell`s: each `c:member` entry is the usual bare business-card stub, and each `c:topic` graph carries that company's service account itself — service name, username, service URI, and password — typed `sa:ServiceAccount`. Google's username is her Gmail address; AT&T's is her mobile phone number, since AT&T accounts are logged into by phone number rather than a separate handle.

<p align="center"><img src="example/images/companies.png" alt="Companies cells"></p>

### Checking Account and Debit Card

Alice has a checking account (and associated debit card) at Citibank. In our example Citibank is compatible with PDN and participates directly, claiming this cell's own topic content — a debit card, a checking account, and Citibank's own record of Alice's online service account — in [graph 76](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-76>). It is colored green because the claimant is Citibank, not Alice. Alice separately self-asserts her own username and password for that same online account in [graph 75](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-75>), and asserts her own notes about Citibank as an institution in [graph 27](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-27>). Alice's own required given-name member entry is [graph 77](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-77>).

<p align="center"><img src="example/images/finances.png" alt="Financial cells"></p>

### Birth Certificate and Driver's License

Alice was born in Texas, and its vital records department issued her a birth certificate. Alice has manually entered the information from her birth certificate into the *Birth Certificate* cell's topic graph ([graph 78](<example/Cells/Government/State/Birth Certificate/Birth Certificate.databook.md#graph-78>)) and has included a scan of her paper birth certificate as content in that cell's Attachments tab (not shown). She recently moved to Paradise, California, and was issued a license by the California DMV. Alice manually entered the information from her plastic license card into the *Drivers License* cell's topic graph ([graph 79](<example/Cells/Government/State/Drivers License/Drivers License.databook.md#graph-79>)) and included a scan of it as content in that cell's Attachments tab (not shown). Each cell's required `member` entry ([graph 24](<example/Cells/Government/State/Birth Certificate/Birth Certificate.databook.md#graph-24>), [graph 15](<example/Cells/Government/State/Drivers License/Drivers License.databook.md#graph-15>)) is just her bare given name.

<p align="center"><img src="example/images/gov-state.png" alt="Government — State cells"></p>

### Passport and Social Security Number

Alice has a social security number (SSN) issued to her by the Social Security Administration, recorded in the *SSN* cell's topic graph ([graph 80](<example/Cells/Government/Federal/SSN/SSN.databook.md#graph-80>)). Similarly, she has a passport issued to her by the US Department of State, recorded in the *Passport* cell's topic graph ([graph 81](<example/Cells/Government/Federal/Passport/Passport.databook.md#graph-81>)). Each cell's required `member` entry ([graph 23](<example/Cells/Government/Federal/SSN/SSN.databook.md#graph-23>), [graph 19](<example/Cells/Government/Federal/Passport/Passport.databook.md#graph-19>)) is just her bare given name.

<p align="center"><img src="example/images/gov-federal.png" alt="Government — Federal cells"></p>

### Current and Previous Homes

Alice used to live in Boston until late 2025, but now lives in Paradise, CA. Both cells are `cat:Home` `TopicCell`s: each `c:member` entry ([graph 13](<example/Cells/Home/Previous/Boston/Boston(home).databook.md#graph-13>), [graph 18](<example/Cells/Home/Paradise/Paradise(home).databook.md#graph-18>)) is the usual bare given-name stub, and each `c:topic` graph carries the actual `residences:Residence` — Boston's in [graph 82](<example/Cells/Home/Previous/Boston/Boston(home).databook.md#graph-82>), Paradise's in [graph 83](<example/Cells/Home/Paradise/Paradise(home).databook.md#graph-83>).

<p align="center"><img src="example/images/home.png" alt="Home cells"></p>

### Possessions

Alice, like everyone, owns (or borrows, or rents) zillions of things. A tiny few of them are described in [graph 22](<example/Cells/Things/Things.databook.md#graph-22>), which concentrates on identity documents: a plastic driver's license card, a health insurance card, and a social security number card. Alice also has a wallet, and keeps some of these cards in it and some separately. She has a vehicle too — see [Vehicles](#vehicles) below. Everything else she owns is out of scope for this example.

Here are a few lines from [graph 22](<example/Cells/Things/Things.databook.md#graph-22>):
```turtle
:Self persona:hasWallet :Alice_Wallet ;
    persona:hasPhysicalCard :Alice_HealthInsuranceCard ;   # carried separately
    persona:hasPhysicalCard :Alice_SSNCard ;               # stored at home
    persona:hasPhysicalCard :Alice_DriversLicense ;        # in wallet
    persona:hasPhysicalCard :Alice_PaymentCard .           # in wallet

:Alice_DriversLicense rdf:type persona:PhysicalDriversLicense ;
    BFO_0000176 :Alice_Wallet .                            # in the wallet

:Alice_PaymentCard rdf:type persona:PhysicalPaymentCard ;
    BFO_0000176 :Alice_Wallet .                            # in the wallet
```

<p align="center"><img src="example/images/things.png" alt="Things cells"></p>

#### Vehicles

Under her *Things* cell, Alice has created a *Vehicles* cell — a purely organizational category node, like *Pets* — and, nested inside it, a cell for her car, named *RAV4* after the car itself (reusing its parent's own `cat:Vehicles` category, the same "child folder reuses its parent's category" pattern the *Ginger* cell already uses under *Pets*). Thanks to `cat:Vehicles`'s own template cell, the *RAV4* cell identifies the car's vehicle type, make and model (real Wikidata individuals — Toyota and the Toyota RAV4), model year, VIN, color, body type, fuel type, drive wheel configuration, current odometer reading, and engine specification as a real `v:Vehicle` individual rather than a bare label ([graph 63](<example/Cells/Things/Vehicles/RAV4/RAV4(vehicles).databook.md#graph-63>)).

Here is a snippet from [graph 63](<example/Cells/Things/Vehicles/RAV4/RAV4(vehicles).databook.md#graph-63>):

```turtle
:Alice_RAV4 rdf:type owl:NamedIndividual ,
                     vehicles:Vehicle ;
    rdfs:label "Alice Walker's RAV4"@en ;

    vehicles:hasVehicleType vehicles:Car ;
    vehicles:hasMake wd:Q53268 ;   # Toyota
    vehicles:hasModel wd:Q819982 ;  # Toyota RAV4
    vehicles:modelYear "2022"^^xsd:gYear ;
    vehicles:vehicleIdentificationNumber "JT3RWRFV1NU012345" ;
    vehicles:color "Silver" ;
    vehicles:bodyType "SUV" ;
    vehicles:fuelType "Gasoline" ;
    vehicles:driveWheelConfiguration "AWD" ;
    vehicles:hasOdometerReading :Alice_RAV4_Odometer ;
    vehicles:hasEngineSpecification :Alice_RAV4_Engine .
```

### Caring for Ginger

Alice also has a cat, Ginger. Under her *Pets* cell she has created a *Ginger* cell ([cell 41](<example/Cells/Pets/Ginger/Ginger(pets).databook.md>)) for this specific pet — reusing its parent's own `cat:Pets` category, and now, thanks to `cat:Pets`'s own template cell, identifying Ginger's name, species (*Felis catus*, an NCBITaxon class IRI), breed (VBO's own "Mixed Breed (Cat)" class), birth date, and current body weight as a real `pets:Pet` individual rather than a bare label ([graph 37](<example/Cells/Pets/Ginger/Ginger(pets).databook.md#graph-37>)).

<p align="center"><img src="example/images/pets.png" alt="Pets cells"></p>

Under [cell 41](<example/Cells/Pets/Ginger/Ginger(pets).databook.md>) is a *Medical* cell ([cell 40](<example/Cells/Pets/Ginger/Medical/Medical.databook.md>)) that Alice created and shared with Paula, who also helps look after Ginger. Its topic is a record of Ginger's medical care (reusing its parent's own `cat:PetsMedical` category) — a completed course of amoxicillin/clavulanate (brand name Clavamox, from Zoetis) and an ongoing daily glucosamine/chondroitin joint supplement ([topic graph 32](<example/Cells/Pets/Ginger/Medical/Medical.databook.md#graph-32>)). The cell contains Alice's claims as a cell `member` in [graph 33](<example/Cells/Pets/Ginger/Medical/Medical.databook.md#graph-33>) and Paula's claims as a cell `member` in [graph 57](<example/Cells/Pets/Ginger/Medical/Medical.databook.md#graph-57>).

Under [cell 41](<example/Cells/Pets/Ginger/Ginger(pets).databook.md>) there is also a *Care & Feeding* cell ([cell 42](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md>)) (`cat:PetsCareAndFeeding`) that Alice also created and also shared with Paula. Its topic records Alice's day-to-day care instructions for Ginger: her feeding schedule and where she sleeps ([topic graph 60](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md#graph-60>)). The cell contains Alice's claims as a cell `member` in [graph 58](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md#graph-58>) and Paula's claims as a cell `member` in [graph 59](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md#graph-59>).

When Alice shares her Medical cell with Paula, the app must decide where to file it in Paula's own tree — see APP-BEHAVIOR.md's [Auto-Filing on Receipt](APP-BEHAVIOR.md#auto-filing-on-receipt) for how that filing heuristic works, using this very cell as its worked example.

### Boston Hub Society

Alice is a member of the Boston Hub Society, an informal professional networking society. In our example BHS has PDN support in its own server, allowing it to participate directly as an `o:Organization` member of this cell, alongside Alice and Bob. Alice maintains her BHS profile in [graph 14](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-14>), Bob, another member, keeps his profile updated ([graph 3](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-03>)), and BHS itself contributes the identity it presents as one of the cell's three parties in [graph 1](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-01>). BHS also claims the cell's one `c:topic` graph — its own organizational profile, recording that the society currently has 80 members and where its website lives ([graph 92](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-92>)). That topic is what makes the cell's derived subject `:BHS` alone, rather than the set of all three member subjects (see integrity.md's Check 18). `cat:Affiliations` is a `c:isTopicCell true` category, so its template cell declares that shape up front — `c:topicGraphShape oshapes:OrganizationShape` — and every cell filed under it is expected to carry a topic of its own (see integrity.md's Check 31); the *Affiliations* scaffold cell above it satisfies that with a deliberately empty topic, since its real content lives down here in this leaf cell instead.

<p align="center"><img src="example/images/affiliations.png" alt="Affiliations cells"></p>

### Planning a Trip with an Agent

Alice is planning a trip with her spouse Dave, and invites her own AI travel agent to help. Under a *Travel* cell (a purely organizational category node, like *Things*) she has created a *Trips* cell (also purely organizational, reusing its parent's own `cat:Travel` category) and, nested inside it, a cell for this specific trip — *Kyoto Trip 2027* (reusing its immediate parent *Trips*'s own `cat:Trips` category, the same "child folder reuses its parent's category" pattern the *Ginger* and *RAV4* cells already use). Alice's travel agent (`a:Agent`) joins this cell as a real member alongside Alice and Dave — not as an invisible tool — giving it its own self-claimed `c:member` graph (see [graph 67](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-67>)) carrying `a:actsFor :Self`. Three distinct members (Self, Dave, and the agent) make this a three-member cell.

The trip itself is backed by three `c:topic` graphs sharing one subject, `:Kyoto_Trip_2027`, one per member, each claimed from a different side: Alice's own basic claim identifying the trip ([graph 69](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-69>)), her agent's own evolving, collaboratively-drafted itinerary ([graph 70](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-70>)), and Dave's own contribution ([graph 91](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-91>)) — reaching `c:topic`'s real upper bound of one value per member (see README.md's [Topic Cell](README.md#topic-cell) section), and extending the same "one topic, multiple claimants" pattern the Medical Appointment cell's two "Med. Appt mt." squares already illustrate (see [Representative Cells](README.md#representative-cells)). The agent's own graph is revised in place turn by turn as Alice chats back and forth with it, rather than replaced by a new graph each time (see APP-BEHAVIOR.md's [Agent Collaboration](APP-BEHAVIOR.md#agent-collaboration)).

<p align="center"><img src="example/images/travel.png" alt="Travel cells"></p>

## Cells Mentioned

A summary of every narratively-illustrated cell under `example/Cells/`, grouped by the narrative subsection above it describes.

| Subsection | Name | Cell DataBook | Subject(s) | Cell Category | Graphs |
|---|---|---|---|---|---|
| Bob and Fred | Bob Johnson | [Bob Johnson(others).databook.md](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md>) {16} | Self, Bob Johnson | `cat:Others` | 2, 4, 8, 12 |
| Bob and Fred | Fred Flintstone | [Fred Flintstone(others).databook.md](<example/Cells/People/Others/Fred Flintstone/Fred Flintstone(others).databook.md>) {17} | Self, Fred Flintstone | `cat:Others` | 29, 31 |
| Taking Care of Sophia | Sophia Walker | [Sophia Walker(immediate-family).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md>) {12} | Sophia Walker | `cat:ImmediateFamily` | 5, 7, 21 |
| Taking Care of Sophia | Health & Wellness | [Health & Wellness.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md>) {13} | Sophia Walker | `cat:HealthWellness` | 17, 35 |
| Taking Care of Sophia | Jane Starostina | [Jane Starostina(primary-care-physician).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Jane Starostina/Jane Starostina(primary-care-physician).databook.md>) {14} | Jane Starostina | `cat:PrimaryCarePhysician` | 25, 34 |
| Taking Care of Sophia | Medical Appointment | [Medical Appointment.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Medical Appointment/Medical Appointment.databook.md>) {15} | Sophia Walker | `cat:MedicalAppointment` | 26, 28, 30 |
| Working for Acme | Alice Walker | [Alice Walker(employees).databook.md](<example/Cells/Work/Acme/Employees/Alice Walker/Alice Walker(employees).databook.md>) {18} | Self | `cat:Employees` | 10 |
| Working for Acme | Paula Walker | [Paula Walker(employees).databook.md](<example/Cells/Work/Acme/Employees/Paula Walker/Paula Walker(employees).databook.md>) {19} | Self, Paula Walker | `cat:Employees` | 6, 20 |
| Service Providers | Google | [Google(companies).databook.md](<example/Cells/Companies/Google/Google(companies).databook.md>) {3} | Alice's Google Account | `cat:Companies` | 16, 73 |
| Service Providers | ATT | [ATT(companies).databook.md](<example/Cells/Companies/ATT/ATT(companies).databook.md>) {2} | Alice's AT&T Account | `cat:Companies` | 11, 74 |
| Checking Account and Debit Card | Citibank | [Citibank(banking-payments).databook.md](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md>) {4} | Self | `cat:BankingPayments` | 27, 75, 76, 77 |
| Birth Certificate and Driver's License | Birth Certificate | [Birth Certificate.databook.md](<example/Cells/Government/State/Birth Certificate/Birth Certificate.databook.md>) {10} | Self | `cat:BirthCertificate` | 24, 78 |
| Birth Certificate and Driver's License | Drivers License | [Drivers License.databook.md](<example/Cells/Government/State/Drivers License/Drivers License.databook.md>) {9} | Self | `cat:DriversLicense` | 15, 79 |
| Passport and Social Security Number | Passport | [Passport.databook.md](<example/Cells/Government/Federal/Passport/Passport.databook.md>) {5} | Self | `cat:Passport` | 19, 81 |
| Passport and Social Security Number | SSN | [SSN.databook.md](<example/Cells/Government/Federal/SSN/SSN.databook.md>) {6} | Self | `cat:SSN` | 23, 80 |
| Current and Previous Homes | Boston | [Boston(home).databook.md](<example/Cells/Home/Previous/Boston/Boston(home).databook.md>) {7} | Self | `cat:Home` | 13, 82 |
| Current and Previous Homes | Paradise | [Paradise(home).databook.md](<example/Cells/Home/Paradise/Paradise(home).databook.md>) {8} | Self | `cat:Home` | 18, 83 |
| Possessions | Things | [Things.databook.md](<example/Cells/Things/Things.databook.md>) {11} | Self | `cat:Things` | 22 |
| Vehicles | RAV4 | [RAV4(vehicles).databook.md](<example/Cells/Things/Vehicles/RAV4/RAV4(vehicles).databook.md>) {44} | Alice's RAV4 | `cat:Vehicles` | 62, 63 |
| Caring for Ginger | Ginger | [Ginger(pets).databook.md](<example/Cells/Pets/Ginger/Ginger(pets).databook.md>) {41} | Ginger | `cat:Pets` | 36, 37 |
| Caring for Ginger | Medical | [Medical.databook.md](<example/Cells/Pets/Ginger/Medical/Medical.databook.md>) {40} | Ginger | `cat:PetsMedical` | 32, 33, 57 |
| Caring for Ginger | Care & Feeding | [Care & Feeding.databook.md](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md>) {42} | Ginger | `cat:PetsCareAndFeeding` | 58, 59, 60 |
| Boston Hub Society | Boston Hub Society | [Boston Hub Society(affiliations).databook.md](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md>) {1} | BHS | `cat:Affiliations` | 1, 3, 14, 92 |
| Planning a Trip with an Agent | Kyoto Trip 2027 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md>) {47} | Kyoto Trip 2027 | `cat:Trips` | 66, 67, 68, 69, 70, 91 |

## Graphs

The graphs in the table below are *about* Alice and claimed *by* Alice. The "Cell DataBook" link jumps straight to each graph's own `### Graph NN` section inside its owning cell-databook file under `example/Cells/`.

| #  | Cell DataBook                                                                          | Category | Key data                                                         | Diagram |
|--- |:--------------------------------------------------------------------------------------|:-------------|:-----------------------------------------------------------------|:--------|
| 10 | [Alice Walker(employees).databook.md](<example/Cells/Work/Acme/Employees/Alice Walker/Alice Walker(employees).databook.md#graph-10>) {18} | `cat:Employees`     | Business card — given name, family name, email, phone, employer  | [view](example/graphs/images/graph-10.png) |
| 11 | [ATT(companies).databook.md](<example/Cells/Companies/ATT/ATT(companies).databook.md#graph-11>) {2}                     | `cat:Companies`    | The ATT cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`), plus an optional organization name and email          | [view](example/graphs/images/graph-11.png) |
| 12 | [Bob Johnson(others).databook.md](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-12>) {16}                     | `cat:Others`       | Alice's 1:1 graph with Bob; social network with Bob as member  | [view](example/graphs/images/graph-12.png)|
| 13 | [Boston(home).databook.md](<example/Cells/Home/Previous/Boston/Boston(home).databook.md#graph-13>) {7}               | `cat:Home` | The Boston cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`)          | [view](example/graphs/images/graph-13.png) |
| 14  | [Boston Hub Society(affiliations).databook.md](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-14>) {1}                     | `cat:Affiliations` | BHS profile: email, phone and current address                    | [view](example/graphs/images/graph-14.png)|
| 15 | [Drivers License.databook.md](<example/Cells/Government/State/Drivers License/Drivers License.databook.md#graph-15>) {9} | `cat:DriversLicense`      | The Drivers License cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`) | [view](example/graphs/images/graph-15.png) |
| 16 | [Google(companies).databook.md](<example/Cells/Companies/Google/Google(companies).databook.md#graph-16>) {3}               | `cat:Companies`    | The Google cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`), plus an optional organization name and email          | [view](example/graphs/images/graph-16.png) |
| 18 | [Paradise(home).databook.md](<example/Cells/Home/Paradise/Paradise(home).databook.md#graph-18>) {8}           | `cat:Home` | The Paradise cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`) | [view](example/graphs/images/graph-18.png) |
| 19 | [Passport.databook.md](<example/Cells/Government/Federal/Passport/Passport.databook.md#graph-19>) {5}             | `cat:Passport`    | The Passport cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`) | [view](example/graphs/images/graph-19.png) |
| 20 | [Paula Walker(employees).databook.md](<example/Cells/Work/Acme/Employees/Paula Walker/Paula Walker(employees).databook.md#graph-20>) {19}                   | `cat:Employees`     | Acme employee graph; company email; works with Paula           | [view](example/graphs/images/graph-20.png)|
| 21 | [Sophia Walker(immediate-family).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md#graph-21>) {12}   | `cat:ImmediateFamily`       | Alice as a family member; family social network with Sophia and Dave                       | [view](example/graphs/images/graph-21.png) |
| 22 | [Things.databook.md](<example/Cells/Things/Things.databook.md#graph-22>) {11}     | `cat:Things`  | Wallet (driver's license + payment card); health ins., SSN card  | [view](example/graphs/images/graph-22.png) |
| 23 | [SSN.databook.md](<example/Cells/Government/Federal/SSN/SSN.databook.md#graph-23>) {6}                     | `cat:SSN`      | The SSN cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`) | [view](example/graphs/images/graph-23.png) |
| 24 | [Birth Certificate.databook.md](<example/Cells/Government/State/Birth Certificate/Birth Certificate.databook.md#graph-24>) {10} | `cat:BirthCertificate`        | The Birth Certificate cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`) | [view](example/graphs/images/graph-24.png) |
| 78 | [Birth Certificate.databook.md](<example/Cells/Government/State/Birth Certificate/Birth Certificate.databook.md#graph-78>) {10} | `cat:BirthCertificate` | Alice's Texas birth certificate — legal names, maiden name; typed `idoc:BirthCertificate` | [view](example/graphs/images/graph-78.png) |
| 79 | [Drivers License.databook.md](<example/Cells/Government/State/Drivers License/Drivers License.databook.md#graph-79>) {9} | `cat:DriversLicense` | California driver's license — legal name, DOB, DL#, expiry, photo; typed `idoc:DriversLicense` | [view](example/graphs/images/graph-79.png) |
| 80 | [SSN.databook.md](<example/Cells/Government/Federal/SSN/SSN.databook.md#graph-80>) {6} | `cat:SSN` | Social security number (SSN) | [view](example/graphs/images/graph-80.png) |
| 81 | [Passport.databook.md](<example/Cells/Government/Federal/Passport/Passport.databook.md#graph-81>) {5} | `cat:Passport` | US passport — legal name, DOB, passport#, issue/expiry, place of birth, gender marker, photo; typed `idoc:Passport` | [view](example/graphs/images/graph-81.png) |
| 82 | [Boston(home).databook.md](<example/Cells/Home/Previous/Boston/Boston(home).databook.md#graph-82>) {7} | `cat:Home` | Previous address — Boston, MA (2020–2025) with temporal interval; typed `residences:Residence` | [view](example/graphs/images/graph-82.png) |
| 83 | [Paradise(home).databook.md](<example/Cells/Home/Paradise/Paradise(home).databook.md#graph-83>) {8} | `cat:Home` | Current address — Paradise, CA (2025–present); typed `residences:Residence` | [view](example/graphs/images/graph-83.png) |
| 29 | [Fred Flintstone(others).databook.md](<example/Cells/People/Others/Fred Flintstone/Fred Flintstone(others).databook.md#graph-29>) {17}                     | `cat:Others`       | Alice's 1:1 graph with Fred; social network with Fred as member  | [view](example/graphs/images/graph-29.png) |
| 33 | [Medical.databook.md](<example/Cells/Pets/Ginger/Medical/Medical.databook.md#graph-33>) {40} | `cat:PetsMedical`     | The Ginger-Medical cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`), plus an optional organization name and email          | [view](example/graphs/images/graph-33.png) |
| 34 | [Jane Starostina(primary-care-physician).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Jane Starostina/Jane Starostina(primary-care-physician).databook.md#graph-34>) {14} | `cat:PrimaryCarePhysician`     | Alice's bare given-name claim — the Jane-Starostina cell's required member entry (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`)          | [view](example/graphs/images/graph-34.png) |
| 35 | [Health & Wellness.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md#graph-35>) {13} | `cat:HealthWellness`     | Alice's bare given-name claim — the Health & Wellness cell's required member entry (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`)          | [view](example/graphs/images/graph-35.png) |
| 36 | [Ginger(pets).databook.md](<example/Cells/Pets/Ginger/Ginger(pets).databook.md#graph-36>) {41} | `cat:Pets`     | The Ginger cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`), plus an optional organization name and email          | [view](example/graphs/images/graph-36.png) |
| 47 | [People.databook.md](<example/Cells/People/People.databook.md#graph-47>) {29} | `cat:People`     | The People cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`), plus an optional organization name and email          | *(todo)* |
| 48 | [Immediate Family.databook.md](<example/Cells/People/Immediate Family/Immediate Family.databook.md#graph-48>) {30} | `cat:ImmediateFamily`     | The Immediate Family cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`), plus an optional organization name and email          | *(todo)* |
| 51 | [Others.databook.md](<example/Cells/People/Others/Others.databook.md#graph-51>) {33} | `cat:Others`     | The Others cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`), plus an optional organization name and email          | *(todo)* |
| 58 | [Care & Feeding.databook.md](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md#graph-58>) {42} | `cat:PetsCareAndFeeding`     | Alice's bare given-name claim — the Ginger-Care & Feeding cell's required member entry (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`)          | [view](example/graphs/images/graph-58.png) |
| 62 | [RAV4(vehicles).databook.md](<example/Cells/Things/Vehicles/RAV4/RAV4(vehicles).databook.md#graph-62>) {44} | `cat:Vehicles`     | The RAV4 cell's required member entry — carries her given name (required by `ContactInfoShape`, `c:memberGraphShape`, since this template is `isTopicCell: true`), plus an optional organization name and email          | [view](example/graphs/images/graph-62.png) |
| 66 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-66>) {47} | `cat:Trips` | Alice's bare given-name claim, extended with her social network link to Dave — one of the Kyoto Trip cell's three required member entries | [view](example/graphs/images/graph-66.png) |
| 75 | [Citibank(banking-payments).databook.md](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-75>) {4} | `cat:BankingPayments` | Alice's own self-asserted claim about her Citibank online service account — username and password as she herself knows them, distinct from Citibank's own record (graph 76) | [view](example/graphs/images/graph-75.png) |
| 77 | [Citibank(banking-payments).databook.md](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-77>) {4} | `cat:BankingPayments` | The Citibank cell's second required member entry, claimed by and about Alice — a minimal given-name stub, so `:Self` is a genuine member of the cell alongside `:Citibank` (graph 27) | [view](example/graphs/images/graph-77.png) |

The following table lists graphs that are *about* Alice but claimed by others.

| #  | Cell DataBook                                                                         | Category | Key data                             | Diagram |
|--- |:-------------------------------------------------------------------------------------|:-------------|:-------------------------------------|:--------|
| 8  | [Bob Johnson(others).databook.md](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-08>) {16}                         | `cat:Others`            | Alice as seen by Bob                 | [view](example/graphs/images/graph-08.png)|
| 76 | [Citibank(banking-payments).databook.md](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-76>) {4} | `cat:BankingPayments` | Citibank's own claimed record about Alice, transmitted from its PDN node — VISA debit card, linked checking account, and online service account | [view](example/graphs/images/graph-76.png) |

The following table lists graphs about other people (Sophia, Dave, Paula and Bob) or organizations (Boston Hub Society) in Alice's own tree. As above, each "Cell DataBook" link jumps to that graph's section inside its owning cell-databook file.

| #  | Cell DataBook                                                                                     | Category | Key data                                                         | Diagram |
|--- |:-------------------------------------------------------------------------------------------------|:-------------|:-----------------------------------------------------------------|:--------|
| 1  | [Boston Hub Society(affiliations).databook.md](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-01>) {1}             | `cat:Affiliations` | BHS's member identity in this cell — name and self-description; claimed by BHS | [view](example/graphs/images/graph-01.png) |
| 2  | [Bob Johnson(others).databook.md](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-02>) {16}                     | `cat:Others`       | Bob's self-claimed Bob persona                                 | [view](example/graphs/images/graph-02.png)|
| 3  | [Boston Hub Society(affiliations).databook.md](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-03>) {1}                     | `cat:Affiliations` | Bob's BHS member persona (name, email, phone, address)          | [view](example/graphs/images/graph-03.png) |
| 92 | [Boston Hub Society(affiliations).databook.md](<example/Cells/Affiliations/Boston Hub Society/Boston Hub Society(affiliations).databook.md#graph-92>) {1} | `cat:Affiliations` | BHS's own organizational profile — its current member count (`o:numMembers` 80) and public website (`o:hasWebsite`) — the cell's manually-added `c:topic`, claimed by BHS | [view](example/graphs/images/graph-92.png) |
| 4  | [Bob Johnson(others).databook.md](<example/Cells/People/Others/Bob Johnson/Bob Johnson(others).databook.md#graph-04>) {16}                 | `cat:Others`       | Alice's notes about Bob; fav drink: oat milk cappuccino         | [view](example/graphs/images/graph-04.png) |
| 5  | [Sophia Walker(immediate-family).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md#graph-05>) {12} | `cat:ImmediateFamily`       | Dave's own self-claimed family persona — the cell's second `member` entry, alongside Alice's own (graph 21)       | [view](example/graphs/images/graph-05.png)|
| 6  | [Paula Walker(employees).databook.md](<example/Cells/Work/Acme/Employees/Paula Walker/Paula Walker(employees).databook.md#graph-06>) {19}           | `cat:Employees`     | Paula as Alice's Acme colleague (Alice-claimed)                | [view](example/graphs/images/graph-06.png)|
| 7  | [Sophia Walker(immediate-family).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Sophia Walker(immediate-family).databook.md#graph-07>) {12} | `cat:ImmediateFamily`       | Sophia as Alice's daughter (Alice-claimed) — the cell's manually-added `c:topic`, using the Contact Info template           | [view](example/graphs/images/graph-07.png)|
| 17 | [Health & Wellness.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Health & Wellness.databook.md#graph-17>) {13} | `cat:HealthWellness`     | Sophia's physical body — height (52 in.), blue eyes, brown hair — as recorded by Alice; linked via `c:topic` (Sophia is the cell's subject, not its member)            | [view](example/graphs/images/graph-17.png) |
| 25 | [Jane Starostina(primary-care-physician).databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Jane Starostina/Jane Starostina(primary-care-physician).databook.md#graph-25>) {14} | `cat:PrimaryCarePhysician`       | Alice's record of Dr. Jane Starostina, Sophia Walker's primary care physician, including her medical specialty (Endocrinology)           | [view](example/graphs/images/graph-25.png)|
| 26 | [Medical Appointment.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Medical Appointment/Medical Appointment.databook.md#graph-26>) {15} | `cat:MedicalAppointment`       | Alice and Dave's shared claims for Sophia's medical appointment — medications, allergies, insurance, PCP reference           | [view](example/graphs/images/graph-26.png)|
| 28 | [Medical Appointment.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Medical Appointment/Medical Appointment.databook.md#graph-28>) {15} | `cat:MedicalAppointment`       | Dave's own self-claimed persona and contact info — one of this cell's two members, alongside Alice (graph 30)           | [view](example/graphs/images/graph-28.png) |
| 30 | [Medical Appointment.databook.md](<example/Cells/People/Immediate Family/Sophia Walker/Health & Wellness/Medical/Provider/Medical Appointment/Medical Appointment.databook.md#graph-30>) {15} | `cat:MedicalAppointment`       | Alice's own self-claimed contact info — the other of this cell's two members, alongside Dave (graph 28)           | [view](example/graphs/images/graph-30.png) |
| 27 | [Citibank(banking-payments).databook.md](<example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md#graph-27>) {4} | `cat:BankingPayments` | Alice's own self-claimed notes about Citibank as an institution, alongside Citibank's own claimed record about her (graph 76) | [view](example/graphs/images/graph-27.png) |
| 31 | [Fred Flintstone(others).databook.md](<example/Cells/People/Others/Fred Flintstone/Fred Flintstone(others).databook.md#graph-31>) {17}                     | `cat:Others`       | Fred's self-claimed Fred persona                                 | [view](example/graphs/images/graph-31.png) |
| 32 | [Medical.databook.md](<example/Cells/Pets/Ginger/Medical/Medical.databook.md#graph-32>) {40} | `cat:PetsMedical`       | Alice's record of her cat Ginger's medications — amoxicillin/clavulanate course, ongoing glucosamine/chondroitin supplement           | [view](example/graphs/images/graph-32.png)|
| 37 | [Ginger(pets).databook.md](<example/Cells/Pets/Ginger/Ginger(pets).databook.md#graph-37>) {41} | `cat:Pets`       | Alice's basic claim identifying Ginger — name, species (Felis catus, NCBITaxon), breed (Mixed Breed (Cat), VBO), birth date, and current body weight — backs the Ginger cell's `subject: ":Ginger"` with a real graph, typed `pets:Pet`           | [view](example/graphs/images/graph-37.png)|
| 57 | [Medical.databook.md](<example/Cells/Pets/Ginger/Medical/Medical.databook.md#graph-57>) {40} | `cat:PetsMedical`       | Paula's own self-claimed given-name claim (required by `ContactInfoShape`) — the cell's second `member` entry after Alice shared it with her, making it a two-member cell — plus an optional organization name and phone           | [view](example/graphs/images/graph-57.png)|
| 59 | [Care & Feeding.databook.md](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md#graph-59>) {42} | `cat:PetsCareAndFeeding`       | Paula's own self-claimed given-name claim (required by `ContactInfoShape`) — the cell's second `member` entry after Alice shared it with her, making it a two-member cell           | [view](example/graphs/images/graph-59.png)|
| 60 | [Care & Feeding.databook.md](<example/Cells/Pets/Ginger/Care & Feeding/Care & Feeding.databook.md#graph-60>) {42} | `cat:PetsCareAndFeeding`       | Alice's day-to-day care and feeding instructions for Ginger — feeding schedule, food, and where she sleeps           | [view](example/graphs/images/graph-60.png)|
| 63 | [RAV4(vehicles).databook.md](<example/Cells/Things/Vehicles/RAV4/RAV4(vehicles).databook.md#graph-63>) {44} | `cat:Vehicles`       | Alice's basic claim identifying her car — vehicle type (Car), make and model (Toyota RAV4, real Wikidata individuals), model year, VIN, color, body type, fuel type, drive wheel configuration, odometer reading, and engine specification — backs the RAV4 cell's `subject: ":Alice_RAV4"` with a real graph, typed `v:Vehicle`           | [view](example/graphs/images/graph-63.png)|
| 67 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-67>) {47} | `cat:Trips` | Alice's travel agent's own self-claimed member graph — typed `a:Agent`, carrying `a:actsFor :Self` | [view](example/graphs/images/graph-67.png)|
| 68 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-68>) {47} | `cat:Trips` | Dave's own self-claimed bare given-name persona — the Kyoto Trip cell's third required member entry, making it a three-member cell | [view](example/graphs/images/graph-68.png)|
| 69 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-69>) {47} | `cat:Trips` | Alice's basic claim identifying the trip itself — backs the Kyoto Trip cell's derived subject `:Kyoto_Trip_2027` with a real graph, distinct from her agent's own contribution (graph 70) | [view](example/graphs/images/graph-69.png)|
| 70 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-70>) {47} | `cat:Trips` | Alice's travel agent's own evolving, collaboratively-drafted itinerary for the trip — a single graph revised in place turn by turn, not replaced each time | [view](example/graphs/images/graph-70.png)|
| 91 | [Kyoto Trip 2027(trips).databook.md](<example/Cells/Travel/Trips/Kyoto Trip 2027/Kyoto Trip 2027(trips).databook.md#graph-91>) {47} | `cat:Trips` | Dave's own contribution to the itinerary — a day trip to Fushimi Inari Taisha and a kaiseki dinner reservation in Gion; the third topic graph, reaching the cell's one-per-member cap | *(todo)* |
| 73 | [Google(companies).databook.md](<example/Cells/Companies/Google/Google(companies).databook.md#graph-73>) {3} | `cat:Companies` | Alice's basic claim about her Google account itself — service name, username (her Gmail address), and password — backs the Google cell's derived subject `:Alice_Google_Account` with a real graph, typed `sa:ServiceAccount` | [view](example/graphs/images/graph-73.png)|
| 74 | [ATT(companies).databook.md](<example/Cells/Companies/ATT/ATT(companies).databook.md#graph-74>) {2} | `cat:Companies` | Alice's basic claim about her AT&T account itself — service name, username (her mobile phone number), service URI, and password — backs the ATT cell's derived subject `:Alice_ATT_Account` with a real graph, typed `sa:ServiceAccount` | [view](example/graphs/images/graph-74.png)|
| 85 | [Companies.databook.md](<example/Cells/Companies/Companies.databook.md#graph-85>) {22} | `cat:Companies` | The Companies scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cells (Google, ATT) instead | [view](example/graphs/images/graph-85.png)|
| 86 | [Banking & Payments Firms(banking-payments).databook.md](<example/Cells/Finances/Banking & Payments Firms/Banking & Payments Firms(banking-payments).databook.md#graph-86>) {24} | `cat:BankingPayments` | The Banking & Payments Firms scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cell (Citibank) instead | [view](example/graphs/images/graph-86.png)|
| 87 | [Home.databook.md](<example/Cells/Home/Home.databook.md#graph-87>) {48} | `cat:Home` | The Home scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cells (Paradise, Boston) instead | [view](example/graphs/images/graph-87.png)|
| 88 | [Pets.databook.md](<example/Cells/Pets/Pets.databook.md#graph-88>) {37} | `cat:Pets` | The Pets scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cell (Ginger) instead | [view](example/graphs/images/graph-88.png)|
| 93 | [Affiliations.databook.md](<example/Cells/Affiliations/Affiliations.databook.md#graph-93>) {21} | `cat:Affiliations` | The Affiliations scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cell (Boston Hub Society) instead | [view](example/graphs/images/graph-93.png)|
| 89 | [Vehicles.databook.md](<example/Cells/Things/Vehicles/Vehicles.databook.md#graph-89>) {43} | `cat:Vehicles` | The Vehicles scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cell (RAV4) instead | [view](example/graphs/images/graph-89.png)|
| 90 | [Trips.databook.md](<example/Cells/Travel/Trips/Trips.databook.md#graph-90>) {46} | `cat:Trips` | The Trips scaffold cell's required `topic` — deliberately empty, since its real content lives in its own leaf cell (Kyoto Trip 2027) instead | [view](example/graphs/images/graph-90.png)|

## Diagrams

`helpers/draw.py` generates a Mermaid (`.mmd`) and PNG diagram for a single embedded graph, given its owning cell DataBook file and its id (or id local-name):

```bash
python3 helpers/draw.py "example/Cells/Finances/Banking & Payments Firms/Citibank/Citibank(banking-payments).databook.md" "graph-76"
python3 helpers/draw.py "example/Cells/Home/Paradise/Paradise(home).databook.md" "graph-18"
```

Both output files are always written to `example/graphs/images/` (must be run from the repo root), keyed by the graph's own id local-name.

**Dependencies** (one-time setup):
```bash
pip install rdflib pyyaml
npm install -g @mermaid-js/mermaid-cli
```

Each diagram shows the `p:Person` individual (yellow), supporting named individuals (white boxes), class labels (plain text), blank-node designator chains, and literal values (green).

## Validation

Validation requires [Apache Jena](https://jena.apache.org/) (`riot`, `shacl`), plus `pyyaml` and `rdflib` for the `helpers/` scripts (`pip install pyyaml rdflib`). All Turtle extraction happens in-repo, through one fence parser (`databook_graphs.iter_graph_blocks()`): `helpers/validate.py` uses it directly, `helpers/extract-graph.py` isolates a single embedded graph, and `helpers/extract-all.py` concatenates every graph in the tree. The [DataBook CLI](https://github.com/kurtcagle/databook) (`databook`; install: `git clone https://github.com/kurtcagle/databook.git && cd databook && npm install && npm install -g .`) is optional — needed only for the Quick check below, never by validation. SHACL shapes remain plain Turtle (`.ttl`).

### Quick check — DataBook syntax (optional, requires the DataBook CLI)

Verify that every DataBook file has valid YAML frontmatter and well-formed block annotations:

```bash
find example -name "*.databook.md" -not -path "*/under-development/*" -print0 | sort -z |
while IFS= read -r -d '' f; do
  databook head "$f" -q > /dev/null || echo "FAIL: $f"
done
```

A file that fails here is likely to yield no Turtle when `helpers/validate.py` extracts it either, causing downstream `riot` or SHACL errors that are harder to trace. (Uses `-print0`/`read -d ''` rather than `for f in $(find ...)` — cell DataBook paths under `example/Cells/` routinely contain spaces, e.g. `Banking & Payments Firms`, which word-splitting would otherwise silently break.)

### Running it

Everything is one script, run from the repo root:

```bash
python3 helpers/validate.py
```

It walks every cell-databook under `example/Cells/` (skipping `under-development/`) and validates
**each cell in isolation from every other cell** — no two cells' data ever reach the same
`shacl validate` call. That isolation is the whole point: every graph re-asserts shared individuals
such as `:Self` under the self-containment convention, so merging all cells into one graph would
union facts that were never meant to co-exist and manufacture violations no real query would ever
see. (README's [Named Graph Scoping](README.md#named-graph-scoping-and-graph-specific-membership) makes the same point about
queries.) The shared foundation and application ontologies it merges in are schema, not another
cell's instance data, so merging those in doesn't break the isolation.

Each cell gets two passes.

**1 — the cell pass.** The cell's whole content at once: every one of its embedded graphs' Turtle,
plus the `c:` triples synthesized from its own `mia.*` frontmatter
(`databook_graphs.process_cell_databook()`). This is validated against the four general shapes
files — `shacl/cell-shacl.ttl` (the cell model itself: `c:category` cardinality, the
`c:TemplateCell`/`c:MemberCell` split, `c:creator`/`c:owner`/`c:member`/`c:topic`, and
`c:SCGraph`'s `c:claimant`/`c:subject`), plus `shacl/persona-shacl.ttl`,
`shacl/organization-shacl.ttl` and `shacl/agent-shacl.ttl` — merged into one shapes graph with
`owl:imports` stripped. The graph Turtle has to be in this data, not just the frontmatter triples:
`:MemberCellShape` and `:SCGraphShape` constrain `c:creator`/`c:owner`/`c:claimant` with
`sh:or ( [sh:class p:Person] [sh:class o:Organization] … )`, and those individuals are typed only
in the graph Turtle.

**2 — the template pass.** Each graph carrying a `template:` value, checked on its own against the
shape that value names. Driven entirely by data already in each cell-databook's own
`mia.member[]`/`mia.topic[]` entries — there is no hand-maintained per-graph command list to keep in
sync. Since `c:template`'s range is `sh:NodeShape` (`cell.ttl`), the value already *names the shape
itself* (e.g. `idocshapes:PassportShape`), with no label-to-shape resolution; the only work left is
locating which physical `*-shacl.ttl` file defines a shape of that name — `pshapes:` shapes are
split across `shacl/persona-shacl.ttl` and `shacl/contactinfo-shacl.ttl` — done via the
`SHAPE_TO_FILE` table in the script. A graph with no `template:` value needs no per-template
validation and is skipped outright.

Every resolved template shape is additionally scoped at runtime so it can't fire on an individual
outside the one graph being checked: every *other* shape co-located in the same physical shapes file
is deactivated for that call, and — for the one class broad enough to risk an incidental same-type
individual within a single isolated graph, `p:Person` (targeted by `ContactInfoShape`) — the
shape is re-targeted (`sh:targetNode`) at only the *substantive* `p:Person` individual(s) actually
present in the graph (one carrying real content, not just the bare `rdf:type` triple the
self-containment convention re-asserts on every referenced individual). Every other template's shape
already targets a narrow, specific document/account class with no such risk, so it keeps its own
original targeting.

The two passes use **different base merges**. `cat-templates.ttl` is in the template pass's base but
deliberately out of the cell pass's, so `cell-shacl`'s `:CellShape` can't fire on the 102
`ctpl:*TemplateCell` individuals — generic class-level content bound to no real person, and not what
a cell-databook's own validation is about. Both bases are built once per run, not once per cell.

Sample output (abridged):

```
OK       example/Cells/Companies/Google/Google(companies).databook.md [cell]
OK       example/Cells/Companies/Google/Google(companies).databook.md graph-16 [pshapes:ContactInfoShape]
OK       example/Cells/Companies/Google/Google(companies).databook.md graph-73 [sashapes:ServiceAccountShape]
OK       example/Cells/Government/Federal/Passport/Passport.databook.md [cell]
OK       example/Cells/Government/Federal/Passport/Passport.databook.md graph-81 [idocshapes:PassportShape]
...

Cells: 46   Checked: 81   Skipped (no template): 9   Violations: 0   Unresolved: 0
```

The script exits non-zero if any cell or checked graph reports a violation (or a `template:` value's
shape has no entry in the `SHAPE_TO_FILE` table), so it doubles as a CI-style gate.

### Merged whole-tree dump

Validation never merges the whole tree, but a few things legitimately need the union — notably
integrity.md's Check 4 ("no orphan Persons"), whose reachability question only makes sense across every
cell at once, and loading the example into a triplestore for ad-hoc SPARQL. To produce it:

```bash
python3 helpers/extract-all.py example > /tmp/mia-data.ttl   # every embedded graph's turtle
python3 helpers/yaml-to-rdf.py . > /tmp/mia-yaml.ttl         # the c: triples from every cell's frontmatter
riot --output=turtle /tmp/mia-data.ttl /tmp/mia-yaml.ttl > /tmp/mia-merged.ttl
```

Do **not** run the general SHACL shapes against this merged file — that is exactly the global-merge
mistake described above.
