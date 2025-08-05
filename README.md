# IIT-PATNA-PROJECT-1
Wi-Fi Password Strength Visualizer is a minimalistic Python utility that lets users estimate the strength of their Wi-Fi passwords through entropy estimation, pattern checking, and an easy-to-understand visual signal.
import re
import matplotlib.pyplot as plt
from math import log2

# List of very common weak passwords
COMMON_PASSWORDS = {"123456", "password", "admin", "qwerty", "letmein", "12345678"}

def calculate_entropy(password):
    """
    Calculates the entropy (bit strength) of a given password
    based on the character sets it uses.
    """
    charset_size = 0

    if re.search(r'[a-z]', password):
        charset_size += 26
    if re.search(r'[A-Z]', password):
        charset_size += 26
    if re.search(r'[0-9]', password):
        charset_size += 10
    if re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
        charset_size += 32

    if charset_size == 0:
        return 0

    entropy_score = len(password) * log2(charset_size)
    return round(entropy_score)


def classify_password(password):
    """
    Classifies password strength as Weak, Moderate, or Strong
    and returns suggestions for improvement.
    """
    score = calculate_entropy(password)
    suggestions = []

    # Check for obvious weaknesses
    if password.lower() in COMMON_PASSWORDS or re.fullmatch(r'(.)\1{5,}', password):
        return "Weak", score, ["Password is too common or uses repeated characters."]

    # Evaluate missing character types
    if len(password) < 8:
        suggestions.append("Password is too short.")
    if not re.search(r'[A-Z]', password):
        suggestions.append("Add uppercase letters.")
    if not re.search(r'[a-z]', password):
        suggestions.append("Add lowercase letters.")
    if not re.search(r'[0-9]', password):
        suggestions.append("Add numbers.")
    if not re.search(r'[!@#$%^&*(),.?\":{}|<>]', password):
        suggestions.append("Add special characters.")

    # Classify based on entropy score
    if score < 40:
        return "Weak", score, suggestions
    elif score < 60:
        return "Moderate", score, suggestions
    else:
        return "Strong", score, suggestions


def show_visual_feedback(score, label):
    """
    Displays a horizontal bar graph of the password's strength.
    """
    color_map = {
        'Weak': 'red',
        'Moderate': 'orange',
        'Strong': 'green'
    }

    plt.figure(figsize=(6, 1))
    plt.barh([0], [score], color=color_map[label])
    plt.xlim(0, 100)
    plt.title(f"Password Strength: {label} ({score} bits entropy)")
    plt.yticks([])
    plt.xlabel("Entropy Score (0–100)")
    plt.tight_layout()
    plt.show()


def main():
    """
    Main function to prompt user input and visualize password strength.
    """
    password = input("🔐 Enter your Wi-Fi password to check its strength: ").strip()
    label, score, tips = classify_password(password)

    show_visual_feedback(score, label)

    if tips:
        print("\n🛠️  Suggestions to strengthen your password:")
        for tip in tips:
            print(f" - {tip}")
    else:
        print("✅ Great! Your password is strong and secure.")


if __name__ == "__main__":
    main()
