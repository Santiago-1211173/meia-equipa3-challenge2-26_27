# AAUTIA Practical Classes — Guide 2

**Instituição:** Instituto Superior de Engenharia do Porto (ISEP) | Universidade Técnica do Porto (UTP)  
**Contacto:** [jmv@isep.ipp.pt](mailto:jmv@isep.ipp.pt)  

---

## 1. Development Environments

The Python programming language is considered the top language for data science and machine learning. You can write the code of the training process of a model using a simple text editor, but you need a development environment to build and execute the code.

> **Remember:** Keep backups and logs of your model training processes, to ensure that you can reuse the models and to improve the reproducibility of the results.

Choose the development environment that suits you best:

a. **Jupyter Notebooks** ([https://docs.jupyter.org/en/latest/](https://docs.jupyter.org/en/latest/))  
b. **Google Colab** ([https://colab.research.google.com/](https://colab.research.google.com/))  
c. **Anaconda** ([https://www.anaconda.com/](https://www.anaconda.com/))  
d. **Spyder** ([https://www.spyder-ide.org/](https://www.spyder-ide.org/))  
e. **JetBrains PyCharm** ([https://www.jetbrains.com/help/pycharm/getting-started.html](https://www.jetbrains.com/help/pycharm/getting-started.html))  
f. **Visual Studio Code** ([https://code.visualstudio.com/docs/languages/python](https://code.visualstudio.com/docs/languages/python))  

### Security Notice

> [!WARNING]
> You are responsible for following security best-practices during your machine learning project.  
> 
> Take note that your development environment may include AI assistants or external applications that store, process, or transmit data in ways that are not fully secure.  
> 
> When handling confidential business data or personally identifiable information, you must disable any software that may have access to your filesystem or to code execution, to ensure regulatory compliance and prevent privacy breaches.  
> 
> For any inquiries, you must consult with your professors and with ISEP’s legal services.

---

## 2. Python Libraries

There are many libraries with open-source Python code available at [PyPI](https://pypi.org/). Instead of writing every algorithm from scratch, you can import the functions of multiple libraries to train machine learning models in a more computationally efficient way.

> **Remember:** Always check the documentation and open issues of each library, to ensure that you only use secure code that has been fully tested.

Start exploring the common libraries, focusing on some useful objects, attributes, and functions:

### a. Numpy
For efficient mathematical operations in the elements of N-dimensional arrays ([Documentation](https://numpy.org/doc/stable/user/whatisnumpy.html)).

```python
# i. Data type of elements
data_type_of_elements = ndarray.dtype

# ii. Shape / Dimensions of the array
size_of_axis0, size_of_axis1, ..., size_of_axisN_1 = ndarray.shape

# iii. Unique elements, inverses, and counts
unique_elements, invs, counts = np.unique(ndarray, return_inverse=True, return_counts=True)

# iv. Means of positive elements along an axis
means_of_positive_elements_of_axis1 = np.mean(ndarray, axis=0, where=(ndarray > 0))
```

### b. Pandas
For data analysis, cleaning, and manipulation in a tabular data format ([Tutorials](https://pandas.pydata.org/docs/getting_started/intro_tutorials/01_table_oriented.html)).

```python
# i. Read CSV file
dataframe = pd.read_csv(filepath, dtype={"col_name": int}, delimiter=",")

# ii. Export to CSV file
dataframe.to_csv(filepath, index=False, encoding="utf-8", sep=",")

# iii. Inspect dataframe structure, statistics, and preview rows
dataframe.info()
dataframe.describe()
print(dataframe.head(20))

# iv. One-hot encoding / dummy variables
columns = pd.get_dummies(dataframe["col_name"], prefix="col_name", dummy_na=False)

# v. Remove a column and concatenate dummy features
removed = dataframe.pop("col_name")
dataframe = pd.concat([dataframe, columns], axis=1)
```

### c. Scikit-learn
For the creation, fine-tuning, and comparison of machine learning models ([Getting Started](https://scikit-learn.org/stable/getting_started.html)).

Useful components:
- `SimpleImputer`
- `IterativeImputer`
- `train_test_split`
- `StratifiedKFold`
- `cross_validate`
- `RandomizedSearchCV`
- `GridSearchCV`
- `StandardScaler`
- `MinMaxScaler`
- `Pipeline`

