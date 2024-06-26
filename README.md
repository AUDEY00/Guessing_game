import socket 
import threading

user_scores = {}

def handle_client(client_socket):
    global user_scores
    
    # dictionary para e convert yung number sa napili ng user ng difficulty
    difficulty_setter = {50: 'easy', 100: 'medium', 500: 'hard'}
     
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
                    break
                elif guess < secret_number:
                    client_socket.send(b"Your guess is too low. Try again.\n")
                else:
                    client_socket.send(b"Your guess is too high. Try again.\n") 
            print("[+] User disconnected. Leaderboard:")
            for name, score_info in sorted(user_scores.items(), key=lambda x: x[1]['score']):
                difficulty_word = difficulty_setter.get(score_info['difficulty'], 'unknown')
                print(f"{name}: Score - {score_info['score']}, Difficulty - {difficulty_word}")
           
def main():  
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind(('192.168.1.11', 8888))
    server_socket.listen(5)
    print("[+] Server is listening for connections...")
    
    while True:
        client_socket, addr = server_socket.accept()
        print("[+] Accepted connection from:", addr)
        client_handler = threading.Thread(target=handle_client, args=(client_socket,))
        client_handler.start()
        print("[+] User disconnected. Leaderboard:")
        for name, score_info in sorted(user_scores.items(), key=lambda x: x[1]['score']):
            print(f"{name}: Score- {score_info['score']}, Difficulty - {score_info['difficulty']}")
    # pang create ng socket object
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
if __name__ == "__main__":
    main()
