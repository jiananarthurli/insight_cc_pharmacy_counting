# Instructions

The input data is stored in ./input/, and the output folder is ./output/ by default.

The program can be called using 
```sh
sh run.sh
```
assuming the input file path is ./input/itcont.txt. 
log.txt is generated when the output file is written. 

The python script can also be called with:
```sh
python ./src/pharmCounting.py input_file_path output_file_path
```

If only top-K number of drug records are needed, the python script can be called with
```sh
python ./src/pharmCounting.py input_file_path output_file_path K
```
where the last parameter specify the number of records in the output. If K is missing, all drug records are written.

# Program ditails

The script reads all the lines in the input file into a record dictionary. The first line is omitted.

The record dictionary has the following structure
```python
records = {
	'drug_name' : {
		'patient_name' : set()
		'total_cost' : float
	}
}
```
```drug_name``` is a unique key for each type of drug. Inside each record,
'patient_name' is a set of unique patient names;
'total_cost' is the total cost of the drug.

After file reading is finished, an array element is generated as
```(drug_name, number_of_distinct_patients, total_cost)```
for each durg record.
The elements are pushed into a minimum heap, where the parent has smaller total_cost than both children. If there is a tie, the element whose drug_name has higher position in ascending alphabetical order will be on the top. If only top K drugs need to be in the output, the heap size is checked whenever a new element is pushed. If max heap size is reached, the record with minimum total_cost will be popped.

Once all the drug records are pushed in th heap, all the min heap elements are popped into a stack, in ascending order. 

The minimum heap is maintained with a heap class.
When a new element is pushed, it is appended to the end of the array, then ```siftUp``` function is called for the new element recursively so that the min heap is maintained.
When an element is popped, it is first swapped with the element in the end of the array and then popped from the array. Then ```siftDown``` function is called for the element at the beginning of the array resursively so that the min heap is maintained.

```siftup``` compares the element with its parent, and swap if necessary.
```siftDown``` compares the element with both children (existence checked first), and swap if necessary.

sorting complexity is nlog(n), n is the number of drugs. If k is specified, the complexity would be nlog(k).

Logging is enabled by default. It can be turned off by passing a parameter when the script is called:
```sh
python ./src/pharmCounting.py input_file_path output_file_path K 0
```
```0``` is the flag for logging.
log.txt records the starting and finishing of file reading. Any anomaly in the file will also be logged, with a line number and system time.

# Classes and functions

```pharmRecord```: record processing class.
	```__init__``` member function that initializes the member variables.
	```__enter__``` member function that initializes the file handlers.
	```__exit__``` member function that recyles the file handlers.
	```log``` internal logging member function.
	```read_input``` member function that reads the file line by line and drug info is written into a record dictionary.
	```sort``` member function that heap-sort the drug records and the result is stored in a stack.
	```write_output``` member function that writes the sorted records in the designated format.
	```records``` member variable that stores the drug info.
	```hp``` member variable for the minimum heap array.
	```stack``` member variable that stores the sorted drug records.
	```k``` member variable that stores the heap size. ```None``` is not specified.
	```logging``` member variable as logging flag.


```pharmHeap```: min heap maintaining class.
	```push``` member function that push a record element into the min heap.
	```pop``` member function that pop the element on top of the min heap.
	```siftUp``` member function that sift up an element recursively.
	```siftDown``` member function that sift down an element recursively.
	```parent``` member function that return the parent index of an element. Returns ```None``` if not existing.	
	```lChild``` member function that return the left child index of an element. Returns ```None``` if not existing.	
	```rChild``` member function that return the right child index of an element. Returns ```None``` if not existing.
	```compare``` member function that compare two elements. Return ```True``` if the left element has smaller total_cost. If there is a tie, return ```True``` if the left drug_name has a higher position in ascending alphabetical order. 

```main```: main function to be called.

