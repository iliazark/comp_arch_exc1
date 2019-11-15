# 075: Computer Architecture - Assignment 1

Ανδρέας Γούλας (9061), Κωνσταντίνος Δουμανίδης (9263)

## Hello world

Το script starter_se.py με τύπο CPU minor χρησιμοποιεί δύο επίπεδα κρυφής
μνήμης και μια walk cache. Οι παράμετροι του συστήματος είναι οι εξής:

Παράμετρος                   |Τιμή         |config.ini
-----------------------------|-------------|-----------------------------------------------
Τύπος CPU                    |Minor        |[system.cpu_cluster.cpus] : type=MinorCPU
Συχνότητα CPU                |4GHz         |[system.cpu_cluster.clk_domain] : clock=250 (ps)
Πλήθος Πυρήνων               |1            |[system.cpu_cluster.cpus] : numThreads=1
Τύπος Μνήμης                 |DDR3 1600 8x8|[system.mem_ctrls1] : tCK=1250 (ps) -> 1600MHz
Πλήθος Καναλιών Μνήμης       |2            |[system.mem_ctrls0]: channels=2
Μέγεθος Μνήμης               |2GB          |[system] : mem_ranges=0:2147483647
Μέγεθος γραμμής Κρυφής Μνήμης|64ΜΒ         |[system] : cache_line_size=64

Διαθέτουμε 2 DRAM Controllers, ένα για κάθε κανάλι.
Το σύστημα έχει CPU Cluster με ένα thread.

### SimpleCPU
Το μοντέλα SimpleCPU περιγράφουν απλούς επεξεργαστές in-order χωρίς να ορίζουν
με λεπτομέρεια την δομή και την λειτουργία του συστήματος. Ανάλογα με τον τύπο
μνήμης που χρησιμοποιείται, υπάρχουν διαφορετικα μοντέλα απλού επεξεργαστή.
Συγκεκριμένα, το μοντέλο TimingSimpleCPU προσομοιώνει με ακρίβεια τις προσβάσεις
στην μνήμη, μοντελοποιώντας τα stall στην κρυφή μνήμη καθώς και την καθυστέρηση
λόγω ουρών. Αντίθετα, ο AtomicSimpleCPU χρησιμοποιεί ένα απλούστερο μοντέλο
μνήμης, το οποίο υπολογίζει προσεγγιστικά τον χρόνο πρόσβασης, χωρίς stalls και
λοιπές καθυστερήσεις.

### Minor CPU
Το μοντέλο επεξεργαστή Minor του gem5 περιγράφει έναν in-order επεξεργαστή. Έχει
ένα σχετικά απλό προκαθορισμένο pipeline που περιγράφεται πιο κάτω και
ρυθμιζόμενες δομές μνήμης όπως και τρόπο λειτουργίας. Το μοντέλο αυτό δεν
υποστηρίζει multithreading επί του παρόντος (όπως και παρατηρήθηκε από το
config.ini από τις δοκιμές μας παραπάνω).

Στάδια Pipeline:
* Fetch1 - Στάδιο ανάγνωσης γραμμών εντολών από την instruction cache
* Fetch2 - Στάδιο σχηματισμού διανυσμάτων εντολών από τις γραμμές του Fetch1
* Decode - Στάδιο αποδόμησης διανύσματος εντολών σε micro-ops
* Execute - Στάδιο εκτέλεσης εντολών και μηχανισμών πρόσβασης στη μνήμη

## Benchmarks

Το πρόγραμμα που προσμοιώθηκε είναι το sort.c με παράμετρο N=1,000.

### MinorCPU, DDR3 1600 8x8

Συχνότητα|Sim Time|Real Time
---------|--------|---------
1GHz     |6.740ms |28.99s
2GHz     |3.385ms |29.21s
4GHz     |1.707ms |29.05s

### TimingSimpleCPU, DDR3 1600 8x8

Συχνότητα|Sim Time|Real Time
---------|--------|---------
1GHz     |20.501ms|11.57s
2GHz     |10.265ms|11.41s
4GHz     |5.147ms |11.60s

### AtomicSimpleCPU, DDR3 1600 8x8

Συχνότητα|Sim Time|Real Time
---------|--------|---------
1GHz     |6.936ms |6.25s
2GHz     |3.468ms |6.39s
4GHz     |1.734ms |6.14s

Παρατηρούμε ότι ο χρόνος προσομοίωσης είναι αντίστρόφως ανάλογος με την
συχνότητα του επξεργαστή. Επίσης, η προσομοίωση με TimingSimpleCPU διαρκεί
περισσότερο χρόνο διότι εμφανίζονται καθυστερήσεις στις προσβάσεις της μνήμης.
Τέλος, τα απλούστερα μοντέλα τρέχουν σε μικρότερο (πραγματικό) χρόνο.

### MinorCPU, 2GHz

Τύπος Μνήμης       |Sim Time|Real Time
-------------------|--------|---------
DDR3 1600 8x8      |3.385ms |30.54s
DDR4 2400 8x8      |3.384ms |30.12s
LPDDR2 S4 1066 1x32|3.395ms |28.53s

### TimingSimpleCPU, 2GHz

Τύπος Μνήμης       |Sim Time|Real Time
-------------------|--------|---------
DDR3 1600 8x8      |10.265ms|11.64s
DDR4 2400 8x8      |10.266ms|11.98s
LPDDR2 S4 1066 1x32|10.274ms|11.61s

### AtomicSimpleCPU, 2GHz

Τύπος Μνήμης       |Sim Time|Real Time
-------------------|--------|---------
DDR3 1600 8x8      |3.468ms |6.69s
DDR4 2400 8x8      |3.468ms |6.52s
LPDDR2 S4 1066 1x32|3.468ms |6.28s

Παρατηρούμε ότι η απόδοση του συστήματος εμφανίζει μία πολύ ασθενή εξάρτηση με
τον τύπο της DRAM που χρησιμοποιείται.

### Πηγές
* [Minor CPU Model](http://www.gem5.org/docs/html/minor.html)
* [Simple CPU Model](http://www.m5sim.org/SimpleCPU)
* [Memory System](http://www.m5sim.org/Memory_System)
