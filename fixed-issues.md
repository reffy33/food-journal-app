# Issues with non UI logic

Issue:

```
ERROR  Database initialization error: [TypeError: Cannot read property 'execAsync' of undefined]
```

Cause of error: the db.withTransactionAsync doesn’t provide the ‘tx’ parameter into the task function
Solution: Use the db object instead

Before:

```jsx
await db.withTransactionAsync(async (tx) => {
  await tx.execAsync(
    ...
  );
});

```

After:

```jsx
await db.withTransactionAsync(async () => {
  await db.execAsync(
    ...
  );
});
```

---

Error:

```
ERROR  Database error: [TypeError: Cannot read property 'rows' of undefined]
```

Cause: Current implementation of execution of SQL queries does not return everything. Also using execAsync for all read and write operations is not optimal solution.

Solution: Rewrite executeSql function, use the runAsync inside, and add helper functions for select many and select one opetaions.

Before:

```jsx
const executeSql = async (query, params = []) => {
  try {
    if (!isInitialized) {
      await initDatabase();
    }

    return await db.withTransactionAsync(async (tx) => {
      return await tx.execAsync(query, params);
    });
  } catch (error) {
    console.error("SQL execution error:", error);
    throw error;
  }
};
```

After:

```jsx
const executeSql = async (query, params = []) => {
  try {
    if (!isInitialized) {
      await initDatabase();
    }

    return await db.runAsync(query, params);
  } catch (error) {
    console.error("SQL execution error:", error);
    throw error;
  }
};

const selectOne = async (query, params = []) => {
  try {
    if (!isInitialized) {
      await initDatabase();
    }

    return await db.getFirstAsync(query, params);
  } catch (error) {
    console.error("SQL execution error:", error);
    throw error;
  }
};

const selectMany = async (query, params = []) => {
  try {
    if (!isInitialized) {
      await initDatabase();
    }

    return await db.getAllAsync(query, params);
  } catch (error) {
    console.error("SQL execution error:", error);
    throw error;
  }
};
```

---

Error: Property ‘Button’ doesn’t exist

Solution: Add the Button import from ‘react-native’

---

Error:

```
ERROR Warning: Error: Element type is invalid: expected a string (for built-in components) or a class/function (for composite components) but got: object.
```

Cause: Expo Camera was not installed and the logic was wrong (probably because the project used the old version of the package)

Solution: Install expo camera package and rewrite the camera permissions logic and use the CameraView component instead of Camera.

# UI issues

Issue: Edit and Delete buttons peaking from the journal entry item.

Solution: Add corresponding border radius to delete button and adjust the height of the block.

```jsx
  hiddenButtons: {
    borderRadius: 8,
    flexDirection: 'row',
    justifyContent: 'flex-end',
    alignItems: 'center',
    height: 110,
  },
    deleteButton: {
    backgroundColor: '#ea4335',
    borderBottomRightRadius: 9,
    borderTopRightRadius: 8
  },
```

---

Issue: Changing the category makes the filter change and vice versa.

Solution: Add a separate state for filter with useState, and update the logic.

```jsx
const [filter, setFilter] = useState('All');

...

const filteredJournals = filter === 'All'
    ? journals
    : journals.filter((item) => item.category === filter);

...

<View style={styles.filterContainer}>
  <Text style={styles.filterLabel}>Filter by:</Text>
  <View style={styles.filterPickerWrapper}>
    <Picker
      selectedValue={filter}
      onValueChange={(itemValue) => setFilter(itemValue)}
    >
      {categories.map((cat) => (
        <Picker.Item key={cat} label={cat} value={cat} />
      ))}
    </Picker>
  </View>
</View>
```

---

Issue: Text inside the filter select component is not shown properly

Solution: Remove the filter picker style that rewrites the height

---

Issue: Only journal entries block can be scrolled, not the whole screen

Solution: Wrap the whole screen with ScrollView and set scrollEnabled to false in the SwipeListView to avoid nesting list errors
