import streamlit as st
import nbformat
from nbconvert import PythonExporter
import subprocess
import tempfile
import os

# Title
st.title("File Upload and Notebook Execution")

# File uploader
uploaded_file = st.file_uploader(
    "Upload your file (only .txt or .json allowed)", 
    type=["txt", "json"]
)

# Run button
if st.button("Run"):
    if uploaded_file:
        # Save uploaded file temporarily
        with tempfile.NamedTemporaryFile(delete=False, suffix=os.path.splitext(uploaded_file.name)[1]) as temp_file:
            temp_file.write(uploaded_file.read())
            uploaded_file_path = temp_file.name
        
        st.success(f"File uploaded successfully: {uploaded_file.name}")
        
        # Execute the Jupyter notebook (test.ipynb)
        notebook_path = "test.ipynb"
        if os.path.exists(notebook_path):
            try:
                # Load the notebook
                with open(notebook_path, "r") as nb_file:
                    notebook = nbformat.read(nb_file, as_version=4)
                
                # Convert to Python code
                exporter = PythonExporter()
                source_code, _ = exporter.from_notebook_node(notebook)

                # Save the source code to a temp .py file
                with tempfile.NamedTemporaryFile(delete=False, suffix=".py") as py_file:
                    py_file.write(source_code.encode("utf-8"))
                    python_script_path = py_file.name
                
                # Modify the Python script to use the uploaded file
                with open(python_script_path, "r") as script_file:
                    script_content = script_file.read()
                modified_content = script_content.replace("input_file_path", f"'{uploaded_file_path}'")
                with open(python_script_path, "w") as script_file:
                    script_file.write(modified_content)

                # Execute the modified Python script
                result = subprocess.run(
                    ["python", python_script_path], 
                    capture_output=True, text=True
                )

                # Display the output
                st.text_area("Execution Output", result.stdout or "No output")
                st.text_area("Execution Errors", result.stderr or "No errors")

            except Exception as e:
                st.error(f"An error occurred while executing the notebook: {e}")
        else:
            st.error("test.ipynb file not found. Please ensure it is in the same directory.")
    else:
        st.error("Please upload a file first.")
