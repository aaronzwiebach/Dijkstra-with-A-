Dijkstra-with-A-Star
================


The following is the explanation and background for the implmenetation of Dijkstra's Algorithm using A-Star that I coded for a 
problem set for the class 6.006 (Introduction to Algorithms):

Premise:
An alien creature is found and we would like to model the alien nervous system.
The nervous system is made up of neurons that are connected by axons.

Each neuron is defined by a tuple with two elements:
1) The ID: the unique number that identifies the neuron
2) Coordinates: Three dimensional Caertesian coordinates that describe the neuron's position in the brain.

Each Axon is defined by a tuple with four elements:
1) Parent Neuron- the ID of the parent neuron
2) Child Neuron- the ID of the Child Neuron
3) Time Travel- the positive number of seconds it takes to travel the axon (i.e.-get from parent neuron to child neuron.)
4) Cognitive Coefficient- a parameter that modulates the strength of the signal


There exists an arbitrary function called Blackbox that we have been given that computes the voltage at any vertex as a function of
of the voltage of the parent neuron and the Cognitive Coefficient of the axon.




Explanation of assignment:

We are told that there is a neuron called Motor "whose output voltage controls all motor functions."

Our goal is to find the the (voltage, Time Travel) of Motor after initializing Neuron 0 (i.e.-the shortest time-travel distance between
Neuron 0 and Motor.)

We will use Dijkstra's Algorithm augmented with A-Star and the admissible heuristic: 3-D Euclidean distance.


NOTE:
DijkstraAstar.py contains only code that I have written.
In that code there are a few methods that are not defined inside DijkstraAstar.py, these are methods that were given to us by the instructors for our 
usage when implementing the algorithm. I have copied all the code below in this document for your perusal, but I have added neccessary comments to DijkstraAstar.py and it can be understood without familiarity of the few methods provided for us.



CODE PROVIDED BY INSTUCTORS:

## There are 5 public functions in the structure.
##
## insert(vertex_id,cost,data) - inserts neuron vertex_id with a specified cost and 
##    supplemental data into the structure
## decrease_key(vertex_id,cost,data) - once neuron vertex_id is inserted into the data
##    structure, you can call decrease_key to reduce the specified cost and replace the
##    supplemental data
## pop() - returns a (vertex_id,cost,data) tuple for the minimum cost neuron
## get_cost(vertex_id) - returns the cost associated with neuron vertex_id.  If vertex_id
##    does not exist in the data structure, this will return None
## is_empty() - returns if the data structure is currently storing any neurons

class hashDictionary():
    def __init__(self):
        self._hashtable = dict()
        self._heap = dict()
        self._numelements = 0
    
    def insert(self,vertex_id,cost,data):
        self._numelements += 1
        self._heap[self._numelements] = [vertex_id,cost,data]
        self._hashtable[vertex_id] = self._numelements
        self._decrease_key(self._numelements)
    
    def _min_heapify(self,heap_pos):
        left = 2*heap_pos
        right= 2*heap_pos+1
        cost = self._heap[heap_pos][1]
        
        swapper = heap_pos

        if((left <= self._numelements) and (self._heap[left][1] < cost)):
            swapper = left
        if((right <= self._numelements) and (self._heap[right][1] < self._heap[swapper][1])):
            swapper = right
        if(swapper != heap_pos):
            self._swap(heap_pos,swapper)
            self._min_heapify(swapper) 

    def _swap(self,heap_pos,swapper):
        temp = self._heap[heap_pos]
        self._heap[heap_pos] = self._heap[swapper]
        self._heap[swapper] = temp
        self._hashtable[self._heap[heap_pos][0]] = heap_pos
        self._hashtable[self._heap[swapper][0]] = swapper


    def decrease_key(self,vertex_id,new_cost,new_data):
        old_cost = self._heap[self._hashtable[vertex_id]][1]
        if(old_cost < new_cost):
            raise Exception("Tried to decrease cost from " + old_cost + " to " + new_cost)
        self._heap[self._hashtable[vertex_id]][1] = new_cost
        self._heap[self._hashtable[vertex_id]][2] = new_data
        self._decrease_key(self._hashtable[vertex_id])
    
    def _decrease_key(self,heap_pos):
        while(heap_pos > 1):
            parent = heap_pos/2
            if(self._heap[heap_pos][1] < self._heap[parent][1]):
                self._swap(heap_pos,parent)
                heap_pos = parent
            else:
                break

    def pop(self):
        if(self._numelements == 0):
            raise Exception("Tried to pop from an empty heap")
        self._swap(1,self._numelements)
        ret = self._heap[self._numelements]
        self._numelements -= 1
        self._min_heapify(1)
        return tuple(ret)
    
    def get_voltage(self,vertex_id):
        if(vertex_id in self._hashtable):
            return self._heap[self._hashtable[vertex_id]][2]
        return None
    
    def get_cost(self,vertex_id):
        if(vertex_id in self._hashtable):
            return self._heap[self._hashtable[vertex_id]][1]
        return None
        
    def is_empty(self):
        return self._numelements == 0             
        
        
## Blackbox function
## Set local_mode to True for local testing.  Set local_mode to False for online testing.
local_mode = False
if(local_mode):
  ## Feel free to put a custom local Blackbox function here.
  def Blackbox(signal,cc):
    return signal + cc
else:
  from BlackboxHolder import Blackbox
