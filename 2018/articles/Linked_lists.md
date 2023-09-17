# Linked lists

A **linked list** is one of the basic data structures, which is a list of values, each of which contains a reference to the next list item (node). This makes it easy to insert and remove items from the list, because the data in the memory should not be stored sequentially. But linked lists consume more memory and have difficulties with accessing data.

Despite the shortcomings, linked lists are the basis for implementing some other types, such as **lists**, **collections** (associative arrays), **queues**, **stacks** etc.

## Implementation in JavaScript

I implemented a small example of a linked list in **JavaScript** (**ES 2015**).

To store the list items, the `Node` class will be used, which contains the stored value and a link to the next node.

```javascript
class Node {

  constructor(value, next) {
    this.value = value;
    this.next = next || null;
  }

}
```

The `List` class contains a reference to the instance of the main (first/head) node. All manipulations with the list come from this node.

```javascript
class List {

  constructor() {
    this._current = null;
    this._head = null;
    this._count = 0;
  }

  add(value) {
    this.addAt(this._count, value);
  }
  
  addAt(index, value) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    if (index == 0) {
      this._head = new Node(value, this._head);
    }
    else {
      let currentIndex = 0;
      let currentNode = this._head;
      let prevNode = currentNode;

      while (currentIndex < index) {
        currentIndex++;
        prevNode = currentNode;
        currentNode = currentNode.next;
      }

      prevNode.next = new Node(value, currentNode);
    }

    this._count++;
  }

  get count() {
    return this._count;
  }

  get current() {
    return this._current;
  }

  indexOf(value) {
    let index = 0;
    let current = this._head;
    
    while (current && current.value != value) {
      current = current.next;
      index++;
    }
    
    return current ? index : -1;
  }
  
  getAt(index) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    let currentIndex = 0;
    let currentNode = this._head;
    
    while (currentNode) {
      if (index == currentIndex) {
        return currentNode;
      }

      currentNode = currentNode.next;
      currentIndex++;
    }
    
    return null;
  }

  moveNext() {
    if (!this._current) {
      this._current = this._head;
    }
    else {
      this._current = this._current.next;
    }

    return this._current != null;
  }

  removeAt(index) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    let currentIndex = 0;
    let currentNode = this._head;
    let prevNode = currentNode;

    while (currentIndex < index) {
      currentIndex++;
      prevNode = currentNode;
      currentNode = currentNode.next;
    }

    prevNode.next = null;
    this._count = currentIndex;

    return currentNode;
  }

  reset() {
    this._current = null;
  }

  toArray() {
    let result = [];
    let current = this._head;

    while (current) {
      result.push(current.value);
      current = current.next;
    }
    
    return result;
  }

}
```

## Performance Tests

Below are some small performance tests for performing various operations with **linked lists** and arrays in **JavaScript**.

> [!WARNING]
> When running tests, the browser may froze.

### Adding items

https://jsfiddle.net/meet_aleksey/7ct61qph/

```javascript
class Node {

  constructor(value, next) {
    this.value = value;
    this.next = next || null;
  }

}

class List {

  constructor() {
    this._head = null;
    this._count = 0;
  }

  add(value) {
    this.addAt(this._count, value);
  }
  
  addAt(index, value) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    if (index == 0) {
      this._head = new Node(value, this._head);
    }
    else {
      let currentIndex = 0;
      let currentNode = this._head;
      let prevNode = currentNode;

      while (currentIndex < index) {
      currentIndex++;
      prevNode = currentNode;
      currentNode = currentNode.next;
      }

      prevNode.next = new Node(value, currentNode);
    }

    this._count++;
  }

  get count() {
    return this._count;
  }

}

let array = [];
let list = new List();
const max = 10000;

let labels = [];
let listLog = [];
let arrayLog = [];

for (let i = 0; i < max; i++) {
  let start = performance.now();
  list.add(i);
  listLog.push(performance.now() - start);
  
  start = performance.now();
  array.push(i);
  arrayLog.push(performance.now() - start);
  
  if ((i % 1000) == 0) {
    labels.push(i);
  }
}

let chart = new Chart($('#chart'), {
  type: 'line',
  data: {
    labels: labels,
    datasets: [{
      label: 'Linked list',
      backgroundColor: 'red',
      borderColor: 'red',
      data: listLog,
      fill: false,
    },
    {
      label: 'Array',
      backgroundColor: 'blue',
      borderColor: 'blue',
      data: arrayLog,
      fill: false,
    }]
  },
  options: {
    responsive: true,
    scales: {
      xAxes: [{
        display: true,
        scaleLabel: {
          display: true,
          labelString: 'Items'
        }
      }],
      yAxes: [{
        display: true,
        scaleLabel: {
          display: true,
          labelString: 'Time (ms)'
        }
      }]
    }
  }
});
```

```html
<canvas id="chart" width="400" height="200"></canvas>
```

### Inserting items randomly

https://jsfiddle.net/meet_aleksey/tdhyLyhq/

```javascript
class Node {

  constructor(value, next) {
    this.value = value;
    this.next = next || null;
  }

}

class List {

  constructor() {
    this._head = null;
    this._count = 0;
  }

  add(value) {
    this.addAt(this._count, value);
  }
  
  addAt(index, value) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    if (index == 0) {
      this._head = new Node(value, this._head);
    }
    else {
      let currentIndex = 0;
      let currentNode = this._head;
      let prevNode = currentNode;

      while (currentIndex < index) {
        currentIndex++;
        prevNode = currentNode;
        currentNode = currentNode.next;
      }

      prevNode.next = new Node(value, currentNode);
    }

    this._count++;
  }

  get count() {
    return this._count;
  }

}
```

```html
<canvas id="chart" width="400" height="200"></canvas>
```

### Finding items

https://jsfiddle.net/meet_aleksey/bvbk48n5/

```javascript
class Node {

  constructor(value, next) {
    this.value = value;
    this.next = next || null;
  }

}

class List {

  constructor() {
    this._head = null;
    this._count = 0;
  }

  add(value) {
    this.addAt(this._count, value);
  }
  
  addAt(index, value) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    if (index == 0) {
      this._head = new Node(value, this._head);
    }
    else {
      let currentIndex = 0;
      let currentNode = this._head;
      let prevNode = currentNode;

      while (currentIndex < index) {
        currentIndex++;
        prevNode = currentNode;
        currentNode = currentNode.next;
      }

      prevNode.next = new Node(value, currentNode);
    }

    this._count++;
  }

  get count() {
    return this._count;
  }

  indexOf(value) {
    let index = 0;
    let current = this._head;
    
    while (current && current.value != value) {
      current = current.next;
      index++;
    }
    
    return current ? index : -1;
  }

}

let array = [];
let list = new List();
const max = 100000;

let labels = [];
let listLog = [];
let arrayLog = [];

for (let i = 0; i < max; i++) {
  list.add(i);
  array.push(i);
}

for (let i = 0, c = max * 0.01; i < c; i++) {
  let randomValue = Math.floor(Math.random() * Math.floor(list.count));
  let start = performance.now();
  list.indexOf(randomValue);
  listLog.push(performance.now() - start);
  
  start = performance.now();
  array.indexOf(randomValue);
  arrayLog.push(performance.now() - start);
  
  if ((i % 100) == 0) {
    labels.push(i);
  }
}

let chart = new Chart($('#chart'), {
  type: 'line',
  data: {
    labels: labels,
    datasets: [{
      label: 'Linked list',
      backgroundColor: 'red',
      borderColor: 'red',
      data: listLog,
      fill: false,
    },
    {
      label: 'Array',
      backgroundColor: 'blue',
      borderColor: 'blue',
      data: arrayLog,
      fill: false,
    }]
  },
  options: {
    responsive: true,
    scales: {
      xAxes: [{
        display: true,
        scaleLabel: {
          display: true,
          labelString: 'Iterations'
        }
      }],
      yAxes: [{
        display: true,
        scaleLabel: {
          display: true,
          labelString: 'Time (ms)'
        }
      }]
    }
  }
});
```

```html
<canvas id="chart" width="400" height="200"></canvas>
```

### Removing items randomly

https://jsfiddle.net/meet_aleksey/p04z8c5q/

```javascript
class Node {

  constructor(value, next) {
    this.value = value;
    this.next = next || null;
  }

}

class List {

  constructor() {
    this._head = null;
    this._count = 0;
  }

  add(value) {
    this.addAt(this._count, value);
  }
  
  addAt(index, value) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    if (index == 0) {
      this._head = new Node(value, this._head);
    }
    else {
      let currentIndex = 0;
      let currentNode = this._head;
      let prevNode = currentNode;

      while (currentIndex < index) {
        currentIndex++;
        prevNode = currentNode;
        currentNode = currentNode.next;
      }

      prevNode.next = new Node(value, currentNode);
    }

    this._count++;
  }

  get count() {
    return this._count;
  }

  removeAt(index) {
    if (index < 0 || index > this.count) {
      throw new Error('The "index" out of range.');
    }

    let currentIndex = 0;
    let currentNode = this._head;
    let prevNode = currentNode;

    while (currentIndex < index) {
      currentIndex++;
      prevNode = currentNode;
      currentNode = currentNode.next;
    }

    prevNode.next = null;
    this._count = currentIndex;

    return currentNode;
  }

}

let array = [];
let list = new List();
const max = 100000;

let labels = [];
let listLog = [];
let arrayLog = [];

for (let i = 0; i < max; ++i) {
  list.add(i);
  array.push(i);
}

for (let i = 0, c = max * 0.01; i < c; i++) {
  let randomIndex = Math.floor(Math.random() * Math.floor(list.count));
  let start = performance.now();
  list.removeAt(randomIndex);
  listLog.push(performance.now() - start);
  
  start = performance.now();
  delete array[randomIndex];
  arrayLog.push(performance.now() - start);
  
  if ((i % 100) == 0) {
    labels.push(i);
  }
}

let chart = new Chart($('#chart'), {
  type: 'line',
  data: {
    labels: labels,
    datasets: [{
      label: 'Linked list',
      backgroundColor: 'red',
      borderColor: 'red',
      data: listLog,
      fill: false,
    },
    {
      label: 'Array',
      backgroundColor: 'blue',
      borderColor: 'blue',
      data: arrayLog,
      fill: false,
    }]
  },
  options: {
    responsive: true,
    scales: {
      xAxes: [{
      display: true,
        scaleLabel: {
          display: true,
          labelString: 'Iterations'
        }
      }],
      yAxes: [{
        display: true,
        scaleLabel: {
          display: true,
          labelString: 'Time (ms)'
        }
      }]
    }
  }
});
```

```html
<canvas id="chart" width="400" height="200"></canvas>
```

---
Aleksey Nemiro  
2018-06-03

https://medium.com/@meet.aleksey/linked-list-e43aff3c303
