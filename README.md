# Setup

The following must be present on an openEHR CDR for this demo to work

- Template ./Botulin Medication.opt with ID "Boutolin medication"
- An EHR with ID "88312867-263e-4c2c-9b6f-9bb59d2537a2" (or change this in App.svelte)
- ECIS Flat/EHRScapre composition endpoint with baseURL="http://localhost:8080/ehrbase/rest/ecis/v1" (or change this in App.svelte)
- A Composition posted at EHRBase with ID "b6142333-74fe-472c-ac30-c90719d1a1c2::1" (or change this in App.svelte)

# Installation

```
npm install
```

# Run server

```
npm run dev
```
