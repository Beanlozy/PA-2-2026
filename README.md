import random

STARTING_DOUBLOONS = 12
STARTING_REPUTATION = 0
WINNING_REPUTATION = 30
FULL_BROADSIDE = 21
RIVAL_STANDS_ON = 17
MAX_SHIPS = 3

SHIP_NAMES = ["Sloop", "Brigantine", "Frigate", "Galleon", "Man-o'-War"]
SHIP_COSTS = [2, 3, 4, 5, 6]
SHIP_REPUTATION = [4, 6, 8, 10, 12]
ENCOUNTERS = ["Merchant Convoy", "Naval Patrol", "Cursed Fog", "Rival Armada", "The Kraken"]
PENALTIES = ["nothing further - the hire fees are simply gone", "pay 3 doubloons in fines",
             "randomly gain or lose 5 reputation", "lose 5 reputation", "lose your ENTIRE doubloon total"]


def get_int_in_range(prompt, low, high):
     while True:
        try:
            number = int(input(prompt).strip())
            if low <= number <= high:
                return number
        except ValueError:
            pass
        print(f"Please enter a whole number from {low} to {high}.")


def get_yes_no(prompt):
    while True:
        answer = input(prompt).strip().lower()
        if answer in ("yes", "y", "no", "n"):
            return answer.startswith("y")
        print("Please answer yes or no.")


def show_status(doubloons, reputation):
    print(f"Doubloons: {doubloons:<16}Reputation: {reputation}")


def get_rank(reputation):
    if reputation >= 20:
        return "Dread Captain"
    if reputation >= 10:
        return "Buccaneer"
    return "Deckhand"


def announce_rank_change(old_reputation, new_reputation):
    new_rank = get_rank(new_reputation)
    if new_rank != get_rank(old_reputation):
        if new_reputation > old_reputation:
            print(f"Word spreads along the docks - you are now a {new_rank}!")
        else:
            print(f"Your name loses its shine - you are back to {new_rank}.")


def check_end_of_run(doubloons, reputation):
    if reputation >= WINNING_REPUTATION:
        return "win"
    if doubloons <= 0:
        return "broke"
    return "continue"


def print_session_summary(start_doubloons, doubloons, reputation):
    print(f"\nThis session: {doubloons - start_doubloons:+} doubloons.   "
          f"Doubloons: {doubloons}   Reputation: {reputation}")


def add_to_log(voyage_log, entry):
    voyage_log.append(f"{len(voyage_log) + 1:>3}. {entry}")


def show_voyage_log(voyage_log):
    print("\n" + "─" * 13 + " Voyage Log " + "─" * 13)
    if not voyage_log:
        print("The log is empty. Go make some history, Captain.")
    for entry in voyage_log:
        print(entry)


def show_main_menu(doubloons, reputation):
    print("\n" + "═" * 40)
    print("  Captain's Cove — Main Menu")
    print("═" * 40)
    print(f"  Doubloons: {doubloons:<14}Reputation: {reputation}")
    print(f"  Title: {get_rank(reputation)}")
    print("─" * 40)
    print("1. Dice Duel\n2. Hire the Fleet\n3. Read the Voyage Log\n4. Leave the Cove")


def show_dice_duel_rules():
    print("HOW DICE DUEL WORKS")
    print("  1. Wager any number of doubloons you hold (at least 1), then roll 2 dice.")
    print("  2. A 6 is worth 10. A 1 is worth 11 unless that would bust you, then 1 ('soft').")
    print("  3. Roll one more die as many times as you like. Go over 21 and you bust.")
    print("  4. The rival rolls last (only if you didn't bust) until it reaches 17 or more.")
    print("  5. Higher total wins, and a rival bust means you win. The rival wins ties,")
    print("     except a FULL BROADSIDE (exactly 21). Win and gain your wager; lose and lose it.")


def roll_die():
    return random.randint(1, 6)


def describe_die(face):
    if face == 6:
        return "6(= 10)"
    if face == 1:
        return "1(= 1 or 11)"
    return str(face)


def add_die(total, soft_aces, face):
    if face == 6:
        total += 10
    elif face == 1:
        total += 11
        soft_aces += 1
    else:
        total += face
    while total > FULL_BROADSIDE and soft_aces > 0:
        total -= 10
        soft_aces -= 1
    return total, soft_aces


def print_roll(text, total, soft_aces):
    soft = " (soft)" if soft_aces > 0 else ""
    print(f"{text:<36}total {total:>2}{soft}")


def open_hand(label):
    first, second = roll_die(), roll_die()
    total, soft_aces = add_die(0, 0, first)
    total, soft_aces = add_die(total, soft_aces, second)
    print_roll(f"{label} {describe_die(first)} and {describe_die(second)}", total, soft_aces)
    return total, soft_aces


def play_captain_hand(): 
    print()
    total, soft_aces = open_hand("You roll:")
    while total < FULL_BROADSIDE and get_yes_no("Roll another? (yes/no): "):
        face = roll_die()
        total, soft_aces = add_die(total, soft_aces, face)
        print_roll(f"You roll: {describe_die(face)}", total, soft_aces)
    if total > FULL_BROADSIDE:
        print(f"Bust! {total} is over 21.")
    elif total == FULL_BROADSIDE:
        print("FULL BROADSIDE - exactly 21!")
    else:
        print(f"You stand on {total}.")
    return total


def play_rival_hand():
    total, soft_aces = open_hand("Rival opens: ")
    while total < RIVAL_STANDS_ON:
        face = roll_die()
        total, soft_aces = add_die(total, soft_aces, face)
        print_roll(f"Rival rolls:  {describe_die(face)}", total, soft_aces)
    if total > FULL_BROADSIDE:
        print(f"The rival busts with {total}!")
    else:
        print(f"Rival stands on {total}.")
    return total


def decide_winner(captain_total, rival_total):
    if captain_total > FULL_BROADSIDE:
        return False
    if rival_total > FULL_BROADSIDE or captain_total > rival_total:
        return True
    return captain_total == rival_total == FULL_BROADSIDE


def dice_duel(doubloons, reputation, voyage_log):
    print("\n" + "─" * 12 + " Dice Duel " + "─" * 12)
    show_dice_duel_rules()
    start_doubloons = doubloons
    while True:
        print()
        show_status(doubloons, reputation)
        wager = get_int_in_range(f"Place your wager (1-{doubloons}): ", 1, doubloons)
        captain_total = play_captain_hand()
        print()
        if captain_total > FULL_BROADSIDE:
            rival_total = 0
            print("The rival doesn't need to roll.")
        else:
            rival_total = play_rival_hand()
        print()
        if decide_winner(captain_total, rival_total):
            doubloons += wager
            result = f"won {wager}"
            print(f"You take the hand! You win {wager} doubloons.")
        else:
            doubloons -= wager
            result = f"lost {wager}"
            print(f"The rival takes the hand. You lose {wager} doubloons.")
        show_status(doubloons, reputation)
        add_to_log(voyage_log, f"Dice Duel - wagered {wager}: you {captain_total} vs rival "
                               f"{rival_total or 'no roll'}, {result}. [Doubloons {doubloons}, Reputation {reputation}]")
        if check_end_of_run(doubloons, reputation) != "continue":
            break
        print()
        if not get_yes_no("Keep playing Dice Duel? (yes/no): "):
            break
    print_session_summary(start_doubloons, doubloons, reputation)
    return doubloons, reputation


def show_fleet_rules():
    print("HOW HIRE THE FLEET WORKS")
    print("  1. Hire 1 to 3 different ships. Every hire fee is paid up front.")
    print("  2. The lookout spots 1 of 5 encounters at random. If you hired the ship it")
    print("     calls for, you earn that ship's reputation. If not, its penalty lands:")
    for i in range(len(ENCOUNTERS)):
        calls_for = f"(calls for {SHIP_NAMES[i]})"
        print(f"       {ENCOUNTERS[i]:<16}{calls_for:<25}{PENALTIES[i]}")
    print("  3. Reputation never drops below 0.")


def show_ships():
    print(f"\n{'Ships for hire:':<25}{'cost':>6}{'rep':>8}")
    for i in range(len(SHIP_NAMES)):
        print(f"  {i + 1}. {SHIP_NAMES[i]:<20}{SHIP_COSTS[i]:>6}{SHIP_REPUTATION[i]:>8}")


def max_affordable_ships(doubloons):
    count = spent = 0
    for cost in SHIP_COSTS:
        if count < MAX_SHIPS and spent + cost <= doubloons:
            spent += cost
            count += 1
    return count


def choose_fleet(doubloons, max_ships):
    while True:
        count = 1 if max_ships == 1 else get_int_in_range(f"Hire how many ships (1-{max_ships})? ", 1, max_ships)
        hired = []
        while len(hired) < count:
            pick = get_int_in_range(f"Ship {len(hired) + 1}: ", 1, len(SHIP_NAMES)) - 1
            if pick in hired:
                print(f"The {SHIP_NAMES[pick]} is already hired - a ship can only be hired once per voyage.")
            else:
                hired.append(pick)
        total_cost = sum(SHIP_COSTS[i] for i in hired)
        if total_cost <= doubloons:
            if total_cost == doubloons:
                print("Careful: this fleet costs every doubloon you hold. Unless this voyage")
                print("takes you to 30 reputation, the run ends when it's over.")
            return hired
        print(f"That fleet costs {total_cost} doubloons, but you only hold {doubloons}. Choose again.")


def apply_penalty(encounter, doubloons, reputation):
    if encounter == 0:
        print("The convoy sails on without you. The hire fees are simply gone.")
        note = "no further penalty"
    elif encounter == 1:
        print("The navy fines you 3 doubloons.")
        doubloons = max(0, doubloons - 3)
        note = "fined 3 doubloons"
    elif encounter == 2:
        change = 5 if random.randint(0, 1) == 0 else -5
        reputation += change
        print(f"The fog plays tricks on your name. {change:+} reputation")
        note = f"{change:+} reputation"
    elif encounter == 3:
        reputation -= 5
        print("The rival armada sends you running. -5 reputation")
        note = "-5 reputation"
    else:
        print(f"The fleet is dragged under. You lose all {doubloons} doubloons.")
        doubloons = 0
        note = "lost every doubloon"
    return doubloons, reputation, note


def hire_the_fleet(doubloons, reputation, voyage_log):
    print("\n" + "─" * 10 + " Hire the Fleet " + "─" * 10)
    show_fleet_rules()
    start_doubloons = doubloons
    while True:
        print()
        show_status(doubloons, reputation)
        max_ships = max_affordable_ships(doubloons)
        if max_ships == 0:
            print("No ship sails for less than 2 doubloons. Win some at the Dice Duel first.")
            break
        show_ships()
        print()
        hired = choose_fleet(doubloons, max_ships)
        fleet_cost = sum(SHIP_COSTS[i] for i in hired)
        hired_names = ", ".join(SHIP_NAMES[i] for i in hired)
        doubloons -= fleet_cost
        print(f"{'Hired: ' + hired_names:<36}-{fleet_cost} doubloons")
        encounter = random.randint(0, len(ENCOUNTERS) - 1)
        print(f"\nThe lookout cries out...   {ENCOUNTERS[encounter].upper()}")
        old_reputation = reputation
        if encounter in hired:
            earned = SHIP_REPUTATION[encounter]
            reputation += earned
            print(f"Your {SHIP_NAMES[encounter]} answers the call.\n+{earned} reputation")
            note = f"{SHIP_NAMES[encounter]} answered, +{earned} reputation"
        else:
            print(f"None of your ships can answer the call (it needed a {SHIP_NAMES[encounter]}).")
            doubloons, reputation, note = apply_penalty(encounter, doubloons, reputation)
        reputation = max(0, reputation)
        print()
        show_status(doubloons, reputation)
        announce_rank_change(old_reputation, reputation)
        add_to_log(voyage_log, f"Hire the Fleet - hired {hired_names} (-{fleet_cost}): {ENCOUNTERS[encounter]}, "
                               f"{note}. [Doubloons {doubloons}, Reputation {reputation}]")
        if check_end_of_run(doubloons, reputation) != "continue":
            break
        print()
        if not get_yes_no("Keep playing Hire the Fleet? (yes/no): "):
            break
    print_session_summary(start_doubloons, doubloons, reputation)
    return doubloons, reputation
    

def main():
    doubloons, reputation, voyage_log = STARTING_DOUBLOONS, STARTING_REPUTATION, []
    print(f"You come ashore at Captain's Cove with {doubloons} doubloons and a name nobody knows yet.")
    print(f"Build {WINNING_REPUTATION} reputation before your doubloons run out.")
    while True:
        show_main_menu(doubloons, reputation)
        choice = input("Enter your choice: ").strip()
        if choice == "1":
            doubloons, reputation = dice_duel(doubloons, reputation, voyage_log)
        elif choice == "2":
            doubloons, reputation = hire_the_fleet(doubloons, reputation, voyage_log)
        elif choice == "3":
            show_voyage_log(voyage_log)
        elif choice == "4":
            print("Fair winds, Captain.")
            return
        else:
            print("Invalid selection. Please choose 1, 2, 3, or 4.")
        result = check_end_of_run(doubloons, reputation)
        if result != "continue":
            print("\n" + "═" * 40)
            if result == "win":
                print("  The harbor master writes your name in the ledger.")
                print(f"  {reputation} reputation - congratulations, {get_rank(reputation)}. You win!")
            else:
                print("  Your purse is empty. With 0 doubloons, the run ends here.")
                print(f"  Final reputation: {reputation}")
            print("═" * 40)
            show_voyage_log(voyage_log)
            return


if __name__ == "__main__":
    try:
        main()
    except (KeyboardInterrupt, EOFError):
        print("\nFair winds, Captain.")
