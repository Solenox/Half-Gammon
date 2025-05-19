# Half-Gammon
//This program manages the moves of checkers in a Backgammon stlye like game, ensuring valid moves based on game rules. If a players checker has been hit and removed, it can re-enter the board following specific rules. The program checks if each move is valid based off different board constraints and the posititon of oppposing checkers' positions before excuting it. If no valid move is possible, the player will be notified.
#include <iostream>
#include <fstream>
#include "mersenne-twister.h"

using namespace std;


const int boardSize = 16; // defines the size of the board
const int checkersPerPlayer = 7; // number of checkers per player

// initializes the board by setting initial checker positions
void initializeBoard(int xBoard[], int oBoard[]) {
	for (int i = 0; i < boardSize + 2; i++) {
		xBoard[i] = 0; // intialize all X checkers to 0
		oBoard[i] = 0; // intialize all O checkers to 0
	}
	xBoard[1] = 5; // places 5 checkers for X and position 1
	xBoard[3] = 2; // places 2 checkers for X and position 3
	oBoard[14] = 2; // places 2 checkers for O and position 14
	oBoard[16] = 5; // places 5 checkers for O and position 16
}

// displays the board with checkers represented by 'X' and 'O'
void displayBoard(int xBoard[], int oBoard[]) {
	for (int height = 7; height >= 1; height--) { // max height of stacks displayed
		cout << "    ";
		for (int pos = 0; pos <= boardSize + 1; pos++) {
			if (xBoard[pos] >= height) {
				cout << "X"; // display 'X' if checker height is sufficient
			} else if (oBoard[pos] >= height) {
				cout << "O"; // display 'O' if checker height is sufficient
			} else {
				cout << " "; // leave space if there is no checker at that height
			}
			if (pos < boardSize + 1) {
				cout << "  "; // add spaces between board positions
			}
		}
		cout << endl;
	}
	// display bottom position numbers
	cout << "    ";
	for (int i = 1; i <= boardSize; i++) {
		cout << i;
		if (i < 10) {
			cout << " "; // space for alignment of numbers
		}
		if (i < boardSize) {
			cout << " "; // space between numbers
		}
	}
	cout << endl;
}

// checks if a move is valid based on game rules
bool isValidMove(int pos, int roll, char player, int xBoard[], int oBoard[]) {
	int* myBoard;
	int* oppBoard;
	int direction;
	int newPos;
	if (pos < 0 || pos > boardSize) {
		return false; // position is out of bounds
	}

	// assigns correct board and movement direction based on the player's piece
	if (player == 'X') {
		myBoard = xBoard;
		oppBoard = oBoard;
		direction = 1; // X moves forward
		newPos = pos + (roll * direction); // calculates new position for O
	} else {
		myBoard = oBoard;
		oppBoard = xBoard;
		direction = -1; // O moves backward
		newPos = pos + (roll * direction); // calculates new position for O
	}

	if (myBoard[pos] == 0) {
		return false; // no checkers at this position
	}

	// checks for valid movement range
	if ((player == 'X' && newPos > boardSize) || (player == 'O' && newPos < 1)) {
		return true; // checker is bearing off
	}

	if (newPos < 1 || newPos > boardSize) {
		return false; // move is out of bounds
	}

	if (oppBoard[newPos] >= 2) {
		return false; // blocked by two or more opponenet checkers
	}

	return true; // valid move
}

// moves a checker from a given position based on the roll
void makeMove(int pos, int roll, char player, int xBoard[], int oBoard[]) {
	int* myBoard;
	int* oppBoard;
	int direction;
	int newPos;

	// determine which board and direction to use based on the current player
	if (player == 'X') {
		myBoard = xBoard;
		oppBoard = oBoard;
		direction = 1;
		newPos = pos + (roll * direction); // X moves forward
	} else {
		myBoard = oBoard;
		oppBoard = xBoard;
		direction = -1;
		newPos = pos + (roll * direction); // O moves forward
	}
	myBoard[pos]--; // remove checker from current position

	// if checker is bearing off, return without placing it on the board
	if((player == 'X' && newPos > boardSize) || (player == 'O' && newPos < 1)) {
		return;
	}

	// if moving to an opponent's occupied position, bump it
	if (oppBoard[newPos] == 1) {
		oppBoard[newPos] = 0; // remove opponents checker
		if (player == 'X') {
			oppBoard[boardSize + 1]++; // send opponents checker to off-board position
		} else {
			oppBoard[0]++; // send opponents checker to off-board position
		}
	}
	myBoard[newPos]++; // place checker in new position
}

// determines if the player has a valid move for a give roll
bool hasValidMove(int roll, char player, int xBoard[], int oBoard[]) {
	int* myBoard;
	if (player == 'X') {
		myBoard = xBoard;
	} else {
		myBoard = oBoard;
	}

	// if a checker is bumped, checks if it can re-enter
	if ((player == 'X' && xBoard[0] > 0) || (player == 'O' && oBoard[boardSize + 1] > 0)) {
		int pos;
		if (player == 'X') {
			pos = 0;
		} else {
			pos = boardSize + 1;
		}

		int newPos;
		if (player == 'X') {
			newPos = roll;
		} else {
			newPos = boardSize + 1 - roll;
		}

		if(newPos >= 1 && newPos <= boardSize) {
			if (player == 'X') {
				return oBoard[newPos] < 2; // checks if position is open for X
			} else {
				return xBoard[newPos] < 2; // checks if position is open for O
			}
		}
		return false; // no valid re-entry
	}

	// checks if any checker can move
	for (int pos = 1; pos <= boardSize; pos++) {
		if (myBoard[pos] > 0 && isValidMove(pos, roll, player, xBoard, oBoard)) {
			return true; // a valid move exists
		}
	}
	return false; // no valid moves exist
}

// counts the number of checkers on the board
int countCheckers(int board[]) {
	int total = 0;
	for (int i = 0; i <= boardSize + 1; i++) {
		total += board[i]; // sum the checkers in each position
	}
	return total;
}

// main game loop, alternating between players
void playGame(int xBoard[], int oBoard[]) {
	char currentPlayer = 'X'; // Player X starts the game
	while (true) {
		displayBoard(xBoard, oBoard); // displays current board
		cout << "It's " << currentPlayer << "'s turn." << endl;

		int roll = chooseRandomNumber(1,6); // roll the dice (random number between 1 and 6)
		cout << "Roll is " << roll << endl;

		bool mustMoveBumped = (currentPlayer == 'X' && xBoard[0] > 0) || (currentPlayer == 'O' && oBoard[boardSize + 1] > 0);

		if (mustMoveBumped) {
			cout << "Bumped checker must move." << endl;
			int newPos;
			if (currentPlayer == 'X') {
				newPos = roll;
			} else {
				newPos = boardSize + 1 - roll;
			}

			if (newPos >= 1 && newPos <= boardSize) {
				bool validMove = false;
				if (currentPlayer == 'X' && oBoard[newPos] < 2) {
					validMove = true;
				} else if (currentPlayer == 'O' && xBoard[newPos] < 2) {
					validMove = true;
				}

				if (validMove) {
					int pos;
					if (currentPlayer == 'X') {
						pos = 0;
					} else {
						pos = boardSize + 1;
					}
					makeMove(pos, roll, currentPlayer, xBoard, oBoard);
				} else {
					cout << "No move possible." << endl;
				}
			} else {
				cout << "No move possible." << endl;
			}

				
		} else {
			if (!hasValidMove(roll, currentPlayer, xBoard, oBoard)) {
				cout << "No move possible." << endl;
			} else {
				int pos;
				do {
					cout << "What position would you like to move (-1 to quit)? ";
					cin >> pos;
					if (pos == -1) {
						return; // exit the game if the user wants
					}
					if (!isValidMove(pos, roll, currentPlayer, xBoard, oBoard)) {
						cout << "Invalid move. Try again." << endl;
					}
				} while (!isValidMove(pos, roll, currentPlayer, xBoard, oBoard)); // keeps asking until a valid move is entered
				makeMove(pos, roll, currentPlayer, xBoard,oBoard); // makes the valid move
			}
		}

		// checks for a win
		if (countCheckers(xBoard) == 0) {
			cout << "Player X Wins!" << endl;
			return; // end game
		}
		if (countCheckers(oBoard) == 0) {
			cout << "Player O Wins!" << endl;
			return; // end game
		}

		//switch players after each turn
		if (currentPlayer == 'X') {
			currentPlayer = 'O';
		} else {
			currentPlayer = 'X';
		}
	}
}

// main function: initializes the game and starts gameplay
int main() 
{
	// Get the number to use as a random seed from the user
	int randSeed;
	cout << "Enter seed: ";
	cin >> randSeed;
	seed(randSeed);

	int xBoard[boardSize + 2] = {0}; // intialize X's board
	int oBoard[boardSize + 2] = {0}; // intialize O's board
	initializeBoard(xBoard, oBoard); // sets the intial board state
	
	playGame(xBoard, oBoard); // starts the game
}
