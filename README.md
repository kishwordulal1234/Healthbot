# Healthbot




import streamlit as st
import google.generativeai as genai
import time

# Configure Google Gemini API
def configure_gemini(api_key):
    genai.configure(api_key=api_key)
    return genai.GenerativeModel('gemini-pro')

def get_medical_analysis(symptoms, model):
    # Prompt engineering for medical analysis
    prompt = f"""
    Based on the following symptoms:
    {', '.join(symptoms)}
    
    Please provide:
    1. Possible diseases/conditions (list top 3)
    2. Confidence percentage for each condition (between 0-100)
    3. Recommended over-the-counter medications or treatments
    4. Whether immediate medical attention is recommended
    
    Format as JSON with keys: diseases (list of dicts with name and confidence), 
    treatments (list), seek_medical_attention (boolean)
    """
    
    response = model.generate_content(prompt)
    return response.text

def main():
    st.set_page_config(page_title="Medical Symptom Analyzer", page_icon="🏥")
    
    st.title("🏥 Medical Symptom Analyzer")
    
    # Disclaimer
    st.warning("""
    ⚠️ DISCLAIMER: This is a prototype medical analysis tool. 
    The results should not be considered as professional medical advice. 
    Always consult with a qualified healthcare provider for proper diagnosis and treatment.
    """)
    
    # API Key Input
    api_key = st.text_input("Enter your Google Gemini API Key:", type="password")
    
    if api_key:
        try:
            model = configure_gemini(api_key)
            
            # Initialize session state for symptoms
            if 'symptoms' not in st.session_state:
                st.session_state.symptoms = []
            
            # Symptom Input Section
            st.subheader("Enter Your Symptoms")
            
            # Display current symptoms
            if st.session_state.symptoms:
                st.write("Current symptoms:")
                for i, symptom in enumerate(st.session_state.symptoms, 1):
                    st.write(f"{i}. {symptom}")
            
            # New symptom input
            new_symptom = st.text_input("Enter a symptom:", key="symptom_input")
            
            if st.button("Add Symptom") and new_symptom:
                if len(st.session_state.symptoms) < 7:
                    st.session_state.symptoms.append(new_symptom)
                    st.experimental_rerun()
                else:
                    st.error("Maximum 7 symptoms allowed!")
            
            if st.button("Clear Symptoms"):
                st.session_state.symptoms = []
                st.experimental_rerun()
            
            # Analysis Section
            if len(st.session_state.symptoms) > 0:
                if st.button("Analyze Symptoms"):
                    with st.spinner("Analyzing symptoms..."):
                        try:
                            # Get medical analysis
                            analysis = get_medical_analysis(st.session_state.symptoms, model)
                            
                            # Display results
                            st.subheader("Analysis Results")
                            
                            # Parse and display the response in a structured way
                            try:
                                import json
                                result = json.loads(analysis)
                                
                                # Display diseases with confidence bars
                                st.write("### Possible Conditions")
                                for disease in result['diseases']:
                                    st.write(f"**{disease['name']}**")
                                    st.progress(disease['confidence'] / 100)
                                    st.write(f"Confidence: {disease['confidence']}%")
                                    st.write("---")
                                
                                # Display treatments
                                st.write("### Recommended Treatments")
                                for treatment in result['treatments']:
                                    st.write(f"- {treatment}")
                                
                                # Medical attention warning
                                if result['seek_medical_attention']:
                                    st.error("⚠️ Based on your symptoms, it is recommended to seek immediate medical attention!")
                                else:
                                    st.info("While these symptoms don't indicate an emergency, consult a healthcare provider if conditions worsen.")
                                
                            except json.JSONDecodeError:
                                st.error("Error parsing the analysis. Please try again.")
                                
                        except Exception as e:
                            st.error(f"An error occurred: {str(e)}")
                            
            else:
                st.info("Please enter at least one symptom to analyze.")
                
        except Exception as e:
            st.error(f"Error configuring API: {str(e)}")
    else:
        st.info("Please enter your Google Gemini API key to proceed.")

if __name__ == "__main__":
    main()
