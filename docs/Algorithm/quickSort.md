```js
function(arr, i, j) {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}
function partition(arr, l, r) {
	// arr[l+1, j] < v ; arr[j+1,i] >= v
  let j = l;
  for(let i = j + 1; i<=r; i++) {
    if(arr[i] < arr[j]) {
      j++;
      swap(arr, i, j);
    }
  }
  swap(arr, l, j);
  return j;
}

function quicksort(arr, l, r) {
  if(l>=r) return;
  let p = partition(arr, l, r);
  quicksort(arr, l, p-1);
 	quicksort(arr, p+1, r); 
}
```

