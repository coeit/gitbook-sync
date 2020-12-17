---
description: an investigation of cassandra array types
---

# cassandra-integration: ARRAY types

### cassandra array types

cassandra offers 2 interesting types of arrays:

#### Set

Cassnadra defines a [set](https://cassandra.apache.org/doc/latest/cql/types.html#sets) Data type. A `set` is a sorted list of unique values of a specific cql-data-type. It supports the following operations:

* inserting
* Adding one or multiple elements \(as this is a set, inserting an already existing element is a no-op\):

  ```text
  UPDATE images SET tags = tags + { 'gray', 'cuddly' } WHERE name = 'cat.jpg';
  ```

* Removing one or multiple elements \(if an element doesn’t exist, removing it is a no-op but no error is thrown\):

  ```text
  UPDATE images SET tags = tags - { 'cat' } WHERE name = 'cat.jpg';
  ```

#### List 

Cassandra also offers a [list](https://cassandra.apache.org/doc/latest/cql/types.html#lists) data type. A `list` is a \(sorted\) collection of non-unique values where elements are ordered by there position in the list. It supports the following operations:

* inserting
* Appending and prepending values to a list:

  ```text
  UPDATE plays SET players = 5, scores = scores + [ 14, 21 ] WHERE id = '123-afde';
  UPDATE plays SET players = 6, scores = [ 3 ] + scores WHERE id = '123-afde';
  ```

* Setting the value at a particular position in the list. This imply that the list has a pre-existing element for that position or an error will be thrown that the list is too small:

  ```text
  UPDATE plays SET scores[1] = 7 WHERE id = '123-afde';
  ```

* Removing an element by its position in the list. This imply that the list has a pre-existing element for that position or an error will be thrown that the list is too small. Further, as the operation removes an element from the list, the list size will be diminished by 1, shifting the position of all the elements following the one deleted:

  ```text
  DELETE scores[1] FROM plays WHERE id = '123-afde';
  ```

* Deleting _all_ the occurrences of particular values in the list \(if a particular element doesn’t occur at all in the list, it is simply ignored and no error is thrown\):

  ```text
  UPDATE plays SET scores = scores - [ 12, 21 ] WHERE id = '123-afde';
  ```

### 

{% hint style="warning" %}
list operations have limitations and performance issues

*  appending and prepending  is not idempotent
* setting and removing specific positions incur an internal _read-before-write_, which is inefficient
{% endhint %}

### cassandra array WHERE read-operations

The only supported operator is the `CONTAINS` operation.

```sql
SELECT * FROM cats WHERE nicknames CONTAINS 'name';
```

### Foreign Key Arrays

Since foreign Key arrays are unique values they seem to be the obvious choice for foreignKey Arrays. We have the possibility to check if a foreign Key exists in the array using `CONTAINS`and we can append or remove foreign keys from the array just fine.

An index created on the column is needed \(`ALLOW FILTERING`also works\)

### Array type attributes

If we want the array to be non-unique we will have to use a list for that. Otherwise the same arguments as for foreign Key arrays apply.

Lists, in theory offer the removal of single elements by index. However it is to be discussed if this is particularly useful, since prior knowledge of the array is implied. Otherwise it is only possible to remove all occurences of a value directly using cql which would circumvent the need for non-unique lists altogether.





