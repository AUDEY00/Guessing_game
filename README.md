import socket 
import threading

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
                if guess < 1 or guess > difficulty:
                    client_socket.send(b"Invalid guess. Please choose a number within the specified range.\n")
                    continue
                attempts += 1
                
                # for checking kung tama ba yung guess ng user o hindi
                if guess == secret_number:
                    user_scores[name]['score'] = min(user_scores[name]['score'], attempts)
                    client_socket.send(f"Congratulations! You guessed the number in {attempts} attempts.\nYour score: {user_scores[name]['score']}\n".encode())
                    
                     with open('leaderboard.txt', 'w') as file:
                        for name, score_info in sorted(user_scores.items(), key=lambda x: x[1]['score']):
                            difficulty_word = difficulty_setter.get(score_info['difficulty'], 'unknown')
                            file.write(f"{name}: Score - {score_info['score']}, Difficulty - {difficulty_word}\n")
                        print("Leaderboard saved successfully.")
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
def main():
    # load user scores from file
    try:
        with open('user_scores.json', 'r') as file:
            user_scores = json.load(file)
    except FileNotFoundError:
        user_scores = {}
    
    # pang create ng socket object
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    # bind the socket to a host and port
    server_socket.bind(('0.0.0.0', 8888))
    
    # listen for incoming connections
    server_socket.listen(5)
    print("[+] Server is listening for connections...")
    while True:
        # accept a new connection
        client_socket, addr = server_socket.accept()
        print("[+] Accepted connection from:", addr)
        
        # start a new thread to handle the client
        client_handler = threading.Thread(target=handle_client, args=(client_socket,))
        client_handler.start()
        
        # display leaderboard when client disconnects
        print("[+] User disconnected. Leaderboard:")
        for name, score_info in sorted(user_scores.items(), key=lambda x: x[1]['score']):
            print(f"{name}: Score - {score_info['score']}, Difficulty - {score_info['difficulty']}")

if __name__ == "__main__":
    main()
