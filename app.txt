import streamlit as st
import difflib
import tempfile
import os

st.set_page_config(
    page_title="Text File Comparison Tool",
    page_icon="📄",
    layout="wide"
)

def highlight_diff(diff):
    """Format diff output with HTML styling for better readability."""
    formatted_diff = []
    for line in diff:
        if line.startswith('+'):
            formatted_diff.append(f'<span style="background-color: #CCFFCC;">{line}</span>')
        elif line.startswith('-'):
            formatted_diff.append(f'<span style="background-color: #FFCCCC;">{line}</span>')
        elif line.startswith('^'):
            formatted_diff.append(f'<span style="background-color: #FFFFCC;">{line}</span>')
        else:
            formatted_diff.append(line)
    return '<br>'.join(formatted_diff)

def save_uploaded_file(uploaded_file):
    """Save an uploaded file to a temporary location and return the path."""
    with tempfile.NamedTemporaryFile(delete=False, suffix='.txt') as temp_file:
        temp_file.write(uploaded_file.getvalue())
        return temp_file.name

def compare_files(file1_content, file2_content):
    """Compare two text files and return the differences."""
    # Split the content into lines
    file1_lines = file1_content.splitlines()
    file2_lines = file2_content.splitlines()
    
    # Get unified diff
    diff = list(difflib.unified_diff(
        file1_lines, 
        file2_lines,
        lineterm='',
        n=3  # Context lines
    ))
    
    # Get HTML diff
    html_diff = difflib.HtmlDiff()
    table = html_diff.make_file(file1_lines, file2_lines)
    
    return diff, table

def main():
    st.title("Text File Comparison Tool")
    st.write("Upload two text files to see the differences between them.")
    
    col1, col2 = st.columns(2)
    
    # Add toggle for input method
    input_method = st.radio(
        "Choose input method:",
        ("Upload Files", "Manual Input"),
        horizontal=True
    )
    
    file1_content = ""
    file2_content = ""
    
    with col1:
        st.subheader("First File")
        if input_method == "Upload Files":
            file1 = st.file_uploader("Upload first text file", type=["txt", "md", "py", "java", "html", "css", "js"])
            if file1 is not None:
                file1_content = file1.getvalue().decode("utf-8")
                st.text_area("Content", file1_content, height=300, disabled=True)
        else:
            file1_content = st.text_area("Enter or paste text for first file", height=300, key="manual_file1")
    
    with col2:
        st.subheader("Second File")
        if input_method == "Upload Files":
            file2 = st.file_uploader("Upload second text file", type=["txt", "md", "py", "java", "html", "css", "js"])
            if file2 is not None:
                file2_content = file2.getvalue().decode("utf-8")
                st.text_area("Content", file2_content, height=300, disabled=True)
        else:
            file2_content = st.text_area("Enter or paste text for second file", height=300, key="manual_file2")
    
    # Check if comparison can be performed
    can_compare = False
    if input_method == "Upload Files":
        can_compare = file1 is not None and file2 is not None
    else:
        can_compare = file1_content.strip() != "" and file2_content.strip() != ""
    
    if can_compare:
        if st.button("Compare Files"):
            st.subheader("Comparison Results")
            
            # Get file contents if using file upload (manual input is already stored)
            if input_method == "Upload Files":
                file1_content = file1.getvalue().decode("utf-8")
                file2_content = file2.getvalue().decode("utf-8")
            
            # Compare files
            diff, html_table = compare_files(file1_content, file2_content)
            
            # Display results
            tab1, tab2 = st.tabs(["Detailed HTML Diff","Simple Diff"])
            
            with tab1:
                if diff:
                    st.markdown("### Side-by-Side Comparison")
                    st.html(html_table)
                else:
                    st.success("The files are identical!")
            


            with tab2:
                if diff:
                    st.markdown("### Changes Found")
                    st.write("Legend:\n- Lines starting with '-' were removed\n- Lines starting with '+' were added\n- Lines starting with '^' indicate changes")
                    st.code(''.join(diff), language="diff", line_numbers=True, wrap_lines=True)
                else:
                    st.success("The files are identical!")
            

            # Calculate statistics
            additions = sum(1 for line in diff if line.startswith('+') and not line.startswith('+++'))
            deletions = sum(1 for line in diff if line.startswith('-') and not line.startswith('---'))
            
            # Display statistics
            st.subheader("Statistics")
            col1, col2, col3 = st.columns(3)
            col1.metric("Additions", additions)
            col2.metric("Deletions", deletions)
            col3.metric("Total Changes", additions + deletions)

if __name__ == "__main__":
    main()