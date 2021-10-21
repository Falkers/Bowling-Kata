using System;
using System.Collections.Generic;

namespace Bowling_Kata
{
    //------------------------------------------------------------------------------------------------------------------
    // Main Class:
    //------------------------------------------------------------------------------------------------------------------

    class BowlingMain
    {
        //-------------------------------------------------------------------------------------------------
        // Options

        public static int numberOfFrames = 10;
        public static int showDebugInfo = 1;    // 0 = off, 1 = scores, 2 = all

        //-------------------------------------------------------------------------------------------------

        public static List<Pin> pinsAll = new List<Pin>();
        public static BowlingSimulation simulation = new BowlingSimulation();
        
        static int scoreTotal;
        static int bonusCounter = 0;

        public static int Main() // Called on Start
        {
            Init();

            for(int i = 0; i < numberOfFrames; i++) 
            {
                simulation.StartFrame(i, false);
                simulation.pinsKnocked.Clear();
            }
            while (simulation.CheckForBonus())
            {
                simulation.StartFrame(numberOfFrames + bonusCounter, true);
                bonusCounter++;
            }
            scoreTotal = simulation.CalculateFinalScore();
            return scoreTotal;
        }

        //------------------------------------------------------------------------
        static void Init()  // Setting up Pins
        {
            // Set up Pins
            for (int i = 0; i < 10; i++)
            {
                Pin pinObj = new Pin(i);
                pinsAll.Add(pinObj);
            }

            // Set Pin connections
            pinsAll[0].SetPinBehind(1, 2);
            pinsAll[1].SetPinBehind(3, 4);
            pinsAll[2].SetPinBehind(4, 5);
            pinsAll[3].SetPinBehind(6, 7);
            pinsAll[4].SetPinBehind(7, 8);
            pinsAll[5].SetPinBehind(8, 9);

            simulation.score = new int[numberOfFrames];
        }
    }

    //------------------------------------------------------------------------------------------------------------------
    // Primary Class: Simulation
    //------------------------------------------------------------------------------------------------------------------

    class BowlingSimulation
    {
        public List<Pin> pinsKnocked = new List<Pin>();

        public bool lastFrameSpare = false;
        public int lastFrameStrike = 0;
        public bool secondLastFrameStrike = false;

        public int[] score;

        //------------------------------------------------------------------------
        public void StartFrame(int number, bool isBonus) // encompasses all functions to simulate a single frame
        {
            BallThrow();                                    // first throw
            if (pinsKnocked.Count == 10)                    // in case of a strike
            {
                CalculateBonus(number, true);                   // true = frame ended
                if (!isBonus) { lastFrameStrike = 2; }      
            }
            else
            {
                CalculateBonus(number, false);              // false = frame is not over yet

                BallThrow();                                // second throw

                CalculateBonus(number, true);               // true = frame ended

                if (pinsKnocked.Count == 10 && !isBonus)    // in case of a spare
                {
                    lastFrameSpare = true;
                }
            }
            if (!isBonus) { score[number] = pinsKnocked.Count; }  //transfers pins to regular score, if this is not a bonus round
            if (BowlingMain.showDebugInfo >= 1) 
            {
                string bonus;
                if      (lastFrameStrike == 2)   { bonus = " (Strike!!)"; }
                else if (lastFrameSpare == true) { bonus = " (Spare!!)"; }
                else                             { bonus = "";}
                Console.WriteLine("Pins hit in Frame " + number + ": " + pinsKnocked.Count + bonus); 
            }
        }

        //------------------------------------------------------------------------
        void CalculateBonus(int frame, bool frameEnded) // Adds any bonus score from a previous strike/spare to the relevant previous score
        {
            if (secondLastFrameStrike)                      // One Bonus (of 2) from a strike 2 frames ago is still active (happens on 2 consecutive strikes)
            {
                score[frame - 2] += pinsKnocked.Count;
                secondLastFrameStrike = false;
            }
            if (lastFrameStrike == 2 && frameEnded)         // Strike Case 1: last frame was strike and this frame is strike
            {
                score[frame - 1] += pinsKnocked.Count;
                lastFrameStrike = 0;
                secondLastFrameStrike = true;                   // this frame ends with 1 Strike-Bonus remaining
            }
            if (lastFrameStrike == 2 && !frameEnded)        // Strike Case 2: last frame was strike and this is the first following throw (no strike)
            {
                lastFrameStrike = 1;                            // 1 Strike-Bonus remaining; no score yet, since pinsKnocked counts total pins
            }
            if (lastFrameStrike == 1 && frameEnded)         // Strike Case 3: last frame was strike and this is the second following throw (thus 1 Bonus done)
            {
                score[frame - 1] += pinsKnocked.Count;
                lastFrameStrike = 0;                            // no Bonus left
            }
            if (lastFrameSpare)                             // Spare
            {
                score[frame - 1] += pinsKnocked.Count;
                lastFrameSpare = false;
            }
        }

        //------------------------------------------------------------------------
        public bool CheckForBonus() // returns true, if a bonus from a strike/spare is still active
        {
            if(lastFrameSpare != false || lastFrameStrike != 0 || secondLastFrameStrike != false){
                return true;
            }else
            {
                return false;
            }
        }

        //------------------------------------------------------------------------
        void BallThrow() // simulates a ball rolling towards pins
        {
            double targetPos = 0.5f;    // where the ball is directed to; when ball moves along x, targetPos is a y coordinate in front of pins
            double currentPos = 0.5f;   // the balls current horizonal(y) position
            int steps = 4;              // the number of iterations for this calculation. Higher numbers are useful to simulate longer distances 
            int firstHit;

            if (BowlingMain.showDebugInfo >= 2) {Console.WriteLine("Ball position at..."); }

            // randomize ball path

            for (int i = 0; i < steps; i++)
            {
                targetPos += (new Random().NextDouble() - 0.5) * 0.8 * (1 - (i/steps));     // the balls target position changes every iteration
                currentPos += (targetPos - 0.5) / steps;                                    // the ball moves towards the target

                if (BowlingMain.showDebugInfo >= 2) { Console.WriteLine("Step " + (i+1) + ": " + currentPos); }
                if (currentPos < 0 || currentPos > 1)                                       // if true, the ball falls into the gutter
                {
                    if (BowlingMain.showDebugInfo >= 2) { Console.WriteLine("Ball hit the gutter"); }
                    return;
                }
            }

            // determine which Pin is hit based on ball position

            if      (currentPos < 0.125) { firstHit = 6; }
            else if (currentPos < 0.250) { firstHit = 3; }
            else if (currentPos < 0.375) { firstHit = 1; }
            else if (currentPos < 0.625) { firstHit = 0; }
            else if (currentPos < 0.750) { firstHit = 2; }
            else if (currentPos < 0.875) { firstHit = 5; }
            else                         { firstHit = 9; }
            
            BowlingMain.pinsAll[firstHit].TryKnockPin(0.9f);
            if (BowlingMain.showDebugInfo >= 2) { Console.WriteLine("Initialy hit Pin " + firstHit); }
        }

        //------------------------------------------------------------------------
        public int CalculateFinalScore() // combines all separately stored scores
        {
            int finalScore = 0;
            for(int i = 0; i < BowlingMain.numberOfFrames; i++)
            {
                finalScore += score[i];
                if (BowlingMain.showDebugInfo >= 1) { Console.WriteLine("Score for Frame " + i + ": " + score[i]); }
            }
            return finalScore;
        }
    }

    //------------------------------------------------------------------------------------------------------------------
    // Sub-Class: Pin
    //------------------------------------------------------------------------------------------------------------------

    public class Pin
    {
        public int id;
        Pin pinBehindLeft = null;
        Pin pinBehindRight = null;

        public Pin(int _id) // set a pin's ID on creation
        {
            id = _id;
        }

        public void SetPinBehind(int left, int right)
        {
            pinBehindLeft = BowlingMain.pinsAll[left];
            pinBehindRight = BowlingMain.pinsAll[right];
        }

        public void TryKnockPin(float chance) // tries to knock over this pin (chance = 0.9 -> 90% success rate)
        {
            double random01 = new Random().NextDouble();
            if (random01 < chance)
            {
                BowlingMain.simulation.pinsKnocked.Add(this);

                //if successful, will also try to knock over pins behind this pin
                if (pinBehindLeft != null && !BowlingMain.simulation.pinsKnocked.Contains(pinBehindLeft))
                {
                    pinBehindLeft.TryKnockPin(0.75f);
                }
                if (pinBehindRight != null && !BowlingMain.simulation.pinsKnocked.Contains(pinBehindRight))
                {
                    pinBehindRight.TryKnockPin(0.75f);
                }
            }
        }
    }
}
