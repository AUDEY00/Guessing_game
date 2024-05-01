import socket 
import threading
import json

# dictionary to store the score of user
user_scores = {}

def handle_client(client_socket):
    global user_scores
    
    # dictionary para e convert yung number sa napili ng user ng difficulty
    difficulty_setter = {50: 'easy', 100: 'medium', 500: 'hard'}
     try:
        while True:
            # for receiving the users data
            data = client_socket.recv(1024).decode()
            if not data:
                break
            name, difficulty = data.split(',')
            difficulty = int(difficulty)
            
            # for initializes to update the user scores
            if name not in user_scores or user_scores[name]['difficulty'] != difficulty:
                user_scores[name] = {'score': float('inf'), 'difficulty': difficulty}
            
            # pang create ng random number based on difficulty
            import random
            secret_number = random.randint(1, difficulty)
            
            attempts = 0
