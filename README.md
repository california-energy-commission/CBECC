# California Building Energy Code Compliance (CBECC) Software 
California Building Energy Code Compliance (CBECC) software for nonresidential, single-family, and multifamily is an open source project that may be used by Code Agencies, Rating Authorities, or Utility Programs in the development of energy codes, standards, or efficiency programs. Architects, engineers, and energy consultants may also use these tools to demonstrate compliance with energy codes or beyond-code programs.

The software's key components are:

- **Graphical User Interface (GUI)** - allows users to enter details about a proposed building's design
- **Ruleset** - a computer-processable form of the building energy code
- **Compliance Manager** - the core of CBECC. Uses the ruleset to assess whether the building complies with the energy code.
- **Connection to the [California Simulation Engine (CSE)](https://github.com/cse-sim/cse) and U.S. Department of Energy's [EnergyPlus](https://energyplus.net/) Simulation Engine** - performs energy simulations to compare proposed building energy consumption to the energy code "budget"
- **Report Generator** - generates forms and other reports to summarize the building's compliance characteristics. Forms may be submitted for building permits, or as documentation for other programs.
- **Application Programming Interface (API) Documentation** - The purpose of this document is to provide information needed to develop software interfaces to the CEC Compliance engine DLL(s).

CBECC includes the Ruleset for California's [Building Energy Efficiency Standards (Title 24)](https://www.energy.ca.gov/programs-and-topics/programs/building-energy-efficiency-standards). The Title 24 Ruleset represents the performance approach for compliance as described in the Nonresidential, Multifamily and Single-Family Residential Alternative Calculation Method (ACM) Reference Manual. It also features an API to allow third party software developers to utilize the functionality of the CBECC Compliance Manager.
