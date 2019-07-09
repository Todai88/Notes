# Shakespeare: A sample service

A sample service that allows a user to search for text in any of Shakespeare's works.

It's made up of two parts:
* A batch component that reads all of Shakespeare's work and writes it to a BigTable. 
    - It runs infrequently, possibly only once (unless new work is identified).
    - Is a MapReduce component comprising of three (3) phases.
        * Mapping: reads Shakespeare's texts and splits them into individual words (faster if performed in parallel).
        * Shuffling: Sorts the tuples (the map) by words.
        * Reduce: A tuple of (word, list of locations) is created.
* The resulting Map of tuples will be written to a BigTable.
* An application front-end that handles end-user requests. 
    - It's always available in all time-zones.
