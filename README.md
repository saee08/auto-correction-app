# auto-correction-app
import streamlit as st
from transformers import pipeline
from difflib import HtmlDiff
import language_tool_python

# Custom CSS for enhanced styling
def load_custom_css():
    st.markdown(
        """
        <style>
        body {
            background-color:rgb(238, 162, 206); /* Light Gray for readability */
            background-image: url('C:/Users/saee santosh pawar/OneDrive/Desktop/autocorrect_api/.streamlit/flat-design-english-school-background_23-2149487419.jpg');
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
            color: #333;
            font-family: Arial, sans-serif;
        }
        .stTextArea, .stTextInput {
            background-color:rgb(243, 14, 128);
            color: #000;
            border-radius: 5px;
            border: 1px solid #ddd;
            padding: 8px;
        }

        .stTextArea textarea, .stTextInput input {
        color:rgb(0, 0, 0)  /* Ensures text color is black */
         }


        .title {
            text-align: center;
            font-size: 34px;
            font-weight: bold;
            margin-bottom: 10px;
            text-shadow: 1px 1px 2px #ccc;
        }
        .footer {
            text-align: center;
            background-color:rgb(2, 12, 12);
            font-size: 14px;
            padding: 10px;
            margin-top: 20px;
            border-top: 1px solid #ddd;
        }
        .error-text {
            color: red;
        }
        </style>
        """,
        unsafe_allow_html=True,
    )

# Initialize grammar correction model using LanguageTool
@st.cache_resource
def initialize_grammar_model():
    try:
        tool = language_tool_python.LanguageTool('en-US')
        return tool
    except Exception as e:
        st.error(f"Error initializing grammar model: {e}")
        return None

# Correct grammar using LanguageTool
def grammar_correction(text, grammar_model):
    try:
        matches = grammar_model.check(text)
        corrected_text = language_tool_python.utils.correct(text, matches)
        return corrected_text
    except Exception as e:
        st.error(f"Grammar correction failed: {e}")
        return text

# Correct text using context-aware correction
def contextual_correction(text, contextual_model):
    try:
        result = contextual_model(f"correct: {text}")
        return result[0]["generated_text"] if result else text
    except Exception as e:
        st.error(f"Contextual correction failed: {e}")
        return text

# Combined correction function
def correct_text(input_text, grammar_model, contextual_model):
    # Step 1: Grammar correction
    grammar_corrected = grammar_correction(input_text, grammar_model)

    # Step 2: Context-aware spelling + grammar correction
    final_corrected = contextual_correction(grammar_corrected, contextual_model)

    return grammar_corrected, final_corrected

# Generate download link for corrected text
def generate_download_link(text, filename="corrected_text.txt"):
    import base64
    b64 = base64.b64encode(text.encode()).decode()
    return f'<a href="data:file/txt;base64,{b64}" download="{filename}">Download Corrected Text</a>'

# Load custom CSS
load_custom_css()

# Streamlit app layout
st.markdown('<div class="title">Intelligent Auto Correction App</div>', unsafe_allow_html=True)

# Initialize models
grammar_model = initialize_grammar_model()
contextual_model = pipeline("text2text-generation", model="facebook/bart-large-cnn")

# Input text
input_text = st.text_area("Enter text for correction:", height=150)

if input_text and grammar_model and contextual_model:
    with st.spinner("Processing your text..."):
        grammar_corrected, final_corrected = correct_text(input_text, grammar_model, contextual_model)

    # Display corrected text
    st.markdown("### Corrected Text:")
    st.text(final_corrected)

    # Display grammar-corrected intermediate step
    st.markdown("### Grammar-Corrected Text:")
    st.text(grammar_corrected)

    # Download corrected text
    st.markdown(generate_download_link(final_corrected), unsafe_allow_html=True)
else:
    st.info("Enter text to start corrections or verify that models are loaded.")

# Footer
st.markdown('<div class="footer">Developed by Saee Santosh Pawar ❤</div>', unsafe_allow_html=True)
