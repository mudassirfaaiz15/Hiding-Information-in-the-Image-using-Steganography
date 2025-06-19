# Hiding-Information-in-the-Image-using-Steganography
# Image Steganography in Python

This project hides secret messages in images using Least Significant Bit (LSB) steganography.

## Features
- Hide secret text messages inside images.
- Extract hidden messages from stego images.
- Uses Python PIL (Pillow) library.

## How to Use
Run `python steganography.py` and follow the instructions.

## Requirements
- Python 3.x
- Pillow (`pip install pillow`)
from PIL import Image

def text_to_binary(message):
    return ''.join(format(ord(char), '08b') for char in message)

def binary_to_text(binary_data):
    chars = [binary_data[i:i+8] for i in range(0, len(binary_data), 8)]
    return ''.join(chr(int(char, 2)) for char in chars)

def hide_data_in_image(input_image_path, output_image_path, secret_message):
    img = Image.open(input_image_path)
    encoded = img.copy()
    width, height = img.size
    binary_message = text_to_binary(secret_message) + '1111111111111110'  # EOF marker
    data_index = 0

    for y in range(height):
        for x in range(width):
            pixel = list(img.getpixel((x, y)))
            for n in range(3):  # R, G, B channels
                if data_index < len(binary_message):
                    pixel[n] = pixel[n] & ~1 | int(binary_message[data_index])
                    data_index += 1
            encoded.putpixel((x, y), tuple(pixel))
            if data_index >= len(binary_message):
                break
        if data_index >= len(binary_message):
            break

    encoded.save(output_image_path)
    print(f"[INFO] Data hidden in {output_image_path}")

def extract_data_from_image(image_path):
    img = Image.open(image_path)
    binary_data = ""
    width, height = img.size

    for y in range(height):
        for x in range(width):
            pixel = img.getpixel((x, y))
            for n in range(3):  # R, G, B channels
                binary_data += str(pixel[n] & 1)
    
    # Split using the EOF marker
    end_marker = '1111111111111110'
    if end_marker in binary_data:
        binary_data = binary_data[:binary_data.find(end_marker)]
    else:
        print("[WARNING] End marker not found. Data may be incomplete.")

    return binary_to_text(binary_data)

# ========== DEMO ==========
if __name__ == "__main__":
    choice = input("Choose: [1] Hide Data  [2] Extract Data\nEnter choice: ")
    
    if choice == '1':
        in_img = input("Enter path of input image: ")
        out_img = input("Enter path for output image: ")
        secret = input("Enter your secret message: ")
        hide_data_in_image(in_img, out_img, secret)
    
    elif choice == '2':
        encoded_img = input("Enter path of encoded image: ")
        hidden_message = extract_data_from_image(encoded_img)
        print("Hidden message:", hidden_message)
    
    else:
        print("Invalid choice.")





