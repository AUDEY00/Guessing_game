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
            while True:
                # receive the user's guess
                guess = int(client_socket.recv(1024).decode())
                attempts += 1
                
                # for checking kung tama ba yung guess ng user o hindi
                if guess == secret_number:
                    user_scores[name]['score'] = min(user_scores[name]['score'], attempts)
                    client_socket.send(f"Congratulations! You guessed the number in {attempts} attempts.\nYour score: {user_scores[name]['score']}\n".encode())
                    break
                elif guess < secret_number:
                    client_socket.send(b"Your guess is too low. Try again.\n")
                else:
                    client_socket.send(b"Your guess is too high. Try again.\n")

             # to displaye the leaderboard when the user disconnect to the server
            print("[+] User disconnected. Leaderboard:")
            for name, score_info in sorted(user_scores.items(), key=lambda x: x[1]['score']):  
                # to get and know the difficulty that user selected
                difficulty_word = difficulty_setter.get(score_info['difficulty'], 'unknown')
                print(f"{name}: Score - {score_info['score']}, Difficulty - {difficulty_word}")
    except Exception as e:
        print("Exception:", e)
    finally:
        client_socket.close()
