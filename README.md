Assignment-4
============
import java.util.*;

public class Board {
	
	private static final int[] RESET = {0,0,0,0,0,0,5,0,3,0,0,0,0,5,0,0,0,0,0,0,0,0,0,0,2,0};
	private static final int BAR  = 25;       // index of the BAR
	private static final int OFF  = 0;        // index of the BEAR OFF
	private static final int INNER_END = 6;   // index for the end of the inner board
	private static final int NUM_PIPS = 26;   // including BAR and BEAR OFF 
	private static final int NUM_PLAYERS = 2; 
	private static final int NUM_CHECKERS = 15; 
	public static final int X_PLAYER_ID = 0;
	public static final int O_PLAYER_ID = 1;

	// FOR DEBUG ONLY: mock resets to test bear off:
	// Backgammon
	//private static final int[] RESET = {14,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1};
	// Gammon
	//private static final int[] RESET = {14,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};
	//Single
	//private static final int[] RESET = {14,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};	
	
	
	private int[][] checkers = new int[NUM_PLAYERS][NUM_PIPS];
			// 2D array of checkers
	        // 1st index: 0 = X_PLAYER_ID's checkers, 1 = O's checkers 
			// 2nd index: number of checkers on each pip, 0 to 25
			// pip 0 is bear off, pip 25 is the bar, pips 1-24 are on the baord

	
	Board () {
		for (int p=0; p<NUM_PLAYERS; p++)  {
			for (int i=0; i<NUM_PIPS; i++)   {
				checkers[p][i] = RESET[i];
			}
		}
		return;
	}
	
	
	public int opposingPlayer (int playerId) {
		int oppPlayerId;
		if (playerId == X_PLAYER_ID) {
			oppPlayerId = O_PLAYER_ID;
		}
		else {
			oppPlayerId = X_PLAYER_ID;
		}
		return oppPlayerId;
	}
	
	
	public int opposingPip (int pip) {
		return BAR-pip;
	}
	
	
	public static void displayChecker (int playerId) {
		if (playerId == X_PLAYER_ID) {
			System.out.print("X");
		}
		else {
			System.out.print("O");
		}
		return;
	}

	
	private void displayPip (int pip) {
		// display the number of checkers on a pip
		int oppPip = opposingPip(pip);  
		
		if  (checkers[X_PLAYER_ID][pip] > 0) {
			displayChecker(X_PLAYER_ID);
			System.out.print(checkers[X_PLAYER_ID][pip] + "  ");
		}
		else
			if (checkers[O_PLAYER_ID][oppPip] > 0) {
				displayChecker(O_PLAYER_ID);
				System.out.print(checkers[O_PLAYER_ID][oppPip] + "  ");
			}
			else {
				System.out.print("|   ");		
			}
		return;
	}	
	
	
	private  void displayOffBoard (int playerId, int pip) {
		// display the number of checkers on the bar or on the bear off
		if  (checkers[playerId][pip] > 0) {
			displayChecker(playerId);
			System.out.print(checkers[playerId][pip] + "  "); 
		}
		else
			System.out.print("    ");			
		return;
	}
	
	
	public void displayBoard (int playerId) {

		// far boards
		if (playerId == X_PLAYER_ID) {
			System.out.println("13--+---+---+---+---18 BAR  19--+---+---+---+---24  OFF");
		}
		else {
			System.out.println("12--+---+---+---+---07 BAR  06--+---+---+---+---01  OFF");
		}
		
		for (int i=13; i<=18; i++) {
			displayPip(i);                // player 1 outer board
		}
		displayOffBoard(X_PLAYER_ID,BAR);       // player 0 bar
		for (int i=19; i<=24; i++) {
			displayPip(i);                // player 1 inner board
		}
		displayOffBoard(O_PLAYER_ID,OFF);         // player 1 bear off
		System.out.println("");

		// separator
		System.out.println("");

		// near boards
		for (int i=12; i>=7; i--) {
			displayPip(i);                 // player 0 outer board
		}
		displayOffBoard(O_PLAYER_ID,BAR);          // player 1 bar
		for (int i=6; i>=1; i--) {
			displayPip(i);                 // player 0 inner board
		}
		displayOffBoard(X_PLAYER_ID,OFF);        // player 0 bear off
		System.out.println("");

		if (playerId == X_PLAYER_ID) {
			System.out.println("12--+---+---+---+---07 BAR  06--+---+---+---+---01  OFF");
		}
		else {
			System.out.println("13--+---+---+---+---18 BAR  19--+---+---+---+---24  OFF");			
		}
		
		System.out.println("");
		
		return;
	}
	
	
	public boolean checkPlay (int playerId, Play play, Dice dice) {
		int startPip, byPips, endPip, oppPlayerId, oppEndPip, lastChecker;
		int[] updateCheckers = new int[NUM_PIPS]; 
		ArrayList<Integer> updateDice = new ArrayList<Integer>(Play.MAX_MOVES);
		Move move;
		boolean valid = true, longBearOff;   // unless proven otherwise
		
		for (int i=0; i<NUM_PIPS; i++) {
			updateCheckers[i] = 0;
		}
		updateDice.add(dice.getDie(0));
		updateDice.add(dice.getDie(1));
		if (dice.getDie(0) == dice.getDie(1)) {  // double 
			updateDice.add(dice.getDie(0));
			updateDice.add(dice.getDie(1));
		}
		
		lastChecker = BAR;
		while (checkers[playerId][lastChecker] == 0) {
			lastChecker--;
		}
		
	    for (int i=0; i<play.length(); i++) {
	    	move = play.getMove(i);
	    	startPip = move.getFromPip();
    		byPips   = move.getByPips();
    		endPip   = startPip - byPips;
    		if (endPip < 0) {
    			endPip = 0;
    			longBearOff = true;
    		}
    		else {
    			longBearOff = false;
    		}
    		oppPlayerId = opposingPlayer(playerId);
    		oppEndPip = opposingPip(endPip);
	    	if ((startPip<OFF) || (startPip>BAR)) {
	    		System.out.println("Invalid pip number: " + startPip);
	    		valid = false;
	    	}
	    	else if (checkers[playerId][startPip] + updateCheckers[startPip] < 1) {
	    		System.out.println("Not enough checkers at pip: " + startPip);
	    		valid = false;
	    	}
	    	else if (!updateDice.contains(byPips)) {
	    		System.out.println("Roll not available: " + byPips);
	    		valid = false;
	    	}
	    	else if ((endPip < BAR) && (endPip > OFF) && (checkers[oppPlayerId][oppEndPip] > 1)) {
	    		System.out.println("Block at pip " + endPip);
	    		valid = false;
	    	}
	    	else if ((endPip == OFF) && (lastChecker > INNER_END)) {
	    		System.out.println("Can't bear off yet");
	    		valid = false;
	    	}
	    	else if (longBearOff && (startPip < lastChecker)) {
	    		System.out.println("Can only bear off the last man with a high roll");
	    		valid = false;
	    	}
	    	else if ((startPip != BAR) && (checkers[playerId][BAR] + updateCheckers[BAR] != 0)) {
	    		System.out.println("Must move the checker on the bar first");
	    		valid = false;
	    	}
	    	else {
	    		updateCheckers[startPip]--;
	    		updateCheckers[endPip]++;
	    		updateDice.remove(updateDice.indexOf(byPips));
	    		if (startPip == lastChecker) {
	    			while (checkers[playerId][lastChecker] + updateCheckers[lastChecker] == 0) {
	    				lastChecker--;
	    			}
	    		}

	    	}
	    }
	    
	    void allPossiblePlays{
			String[] allPossiblePlays;
			System.out.print("allPossiblePlays.toString" + allPossiblePlays.toString());
			if (attemptedMoves==0){
				if(Move.NUM_CHECKERS() > 0){
					Move.BgTwoPlayer();
					doPlay(Board, move);
				}
			if ( Dice == 4 || dice[ Dice ] == 0 ) {
				Move(Board,move);
				return;
			}
				     	
		return valid;
	}
	
	
	public boolean doPlay (int playerId, Play play) {
		// move checkers 
		int startPip, endPip, oppLastChecker;
		int oppPlayerId, oppPip;
		Move move;
		boolean finished;
		
		oppPlayerId = opposingPlayer(playerId);
	
	    for (int i=0; i<play.length(); i++) {
	    	move = play.getMove(i);
	    	startPip = move.getFromPip();
	    	checkers[playerId][startPip] -= 1;
    		endPip   = startPip - move.getByPips();
     		if (endPip < OFF)	{							// check for long bear offs
	    		endPip = OFF;   
     		}
	    	checkers[playerId][endPip] += 1;
	    	oppPip = opposingPip(endPip);
	    	if ((checkers[oppPlayerId][oppPip]==1) && (endPip!=BAR) && (endPip!=OFF))	{	// check for HIT
	    		checkers[oppPlayerId][oppPip] -= 1;
	    		checkers[oppPlayerId][BAR] += 1;
	    	}
	    }	
	    
	    if (checkers[playerId][OFF] == NUM_CHECKERS) {
	    	displayChecker(playerId);
	    	System.out.print(" WINS ");
	    	oppLastChecker = BAR;
	    	while (checkers[oppPlayerId][oppLastChecker] == 0) {
	    		oppLastChecker--;
	    	}
	    	if (oppLastChecker >= opposingPip(INNER_END)) {
	    		System.out.println("A BACKGAMMON!!!");
	    	}
	    	else if (oppLastChecker > INNER_END) {
	    		System.out.println("A GAMMON!!");
	    	}
	    	else {
	    		System.out.println("A SINGLE!");
	    	}
	    	finished = true;
	    }
	    else {
	    	finished = false;
	    }
	    
	    return finished;
	}
	void allPossiblePlays(int[] Board, int[] dice, int Dice, Move move ) {
	   
	 	//the 'to' location based on 'from' location and dice roll
	    int i, to;
	    int die = dice[Dice];
	    int attemptedMoves = 0;
	    int high_X_PLAYER_ID = X_PLAYER_ID(Board);
	    // sweep the board, i iterating all possible board locations
	    for (i=0;i<=24;i++) {
	      if (Board[i]>0) {
	        // determine the location to move to
	        to = endPip(i,die, high_X_PLAYER_ID);

	        // check that the move is valid
	        boolean isValid = ( to <= OFF && valid( Board, dice[Dice], i, to ) );
	        if ( isValid ) {
	          // value copy the board to avoid recursion level cross talk
	          int [] boardcopy = new int[RESET];
	          System.arraycopy( Board, 0, boardcopy, 0, RESET );
	          // make the move on the copied board
	          Move(boardcopy, i, to);

	          Move movecopy = move.getPip();
	          movecopy.add(i, to);

	          allPossiblePlays(boardcopy, dice, Dice+1, movecopy);
	          attemptedMoves ++;
	        }
	      }
	    }
}
}
