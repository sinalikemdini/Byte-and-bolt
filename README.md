'''GUI based hire system developed for the Byte and bolt Tech hire.
Name- Sinali Kemdini
Date- 27/07/2026'''

# The program is managed by store staff and allows them to:
# Add hire records.
# View current hire records.
# Search for records and delete them.
# Automatically generate and display receipt numbers.

# Program validation rules requirements:
# Customer name should only contain letters and should be 12 characters or fewer.
# An item must be selected.
# The quantity should be a whole number from 1 to 20.
# The date must be today or a future date.

import tkinter as tk
from tkinter import ttk,messagebox
from PIL import Image,ImageTk
import json
import os
import random
from tkcalendar import Calendar
from datetime import date,datetime

# The colour constants I used for my program.
# Using colour constants made it easier for me to do the coding as I don't need to type the code of the colour always where no errors would be happening.
TEAL="#00687a"
NAVY_BLUE="#020c45"
BG_CL="#dedede"
LIGHT_TEAL="#0097b2"
WHITE="#ffffff"
BLACK="#000000"

# I used a constant for the items that can be selected so it is easier for me because if I have to type it out it will be confusing for the people who read it also.
ACCESSORIES=["Gaming Mouse","Keyboard","Headsets","Monitor","Webcam","USB Hub"]
# I used icons for the items so the item selection is easier to understand.
# I used a different constant for icons so I can use it for the places I need the buttons with icons.
ACCESSORIES_ICONS=["🖱 Gaming Mouse","⌨️ Keyboard","🎧 Headsets","🖥 Monitor","📷 Webcam","🔌 USB Hub"]
FONT_LBL=("arial",16) # I used this constant so it is easier to reuse the same label font.
QTY_MIN=1 # Minimum quantity that can be selected from the spinbox.
QTY_MAX=20 # Maximum quantity that can be selected from the spinbox.
NAME_MAX=15 # Maximum amount of characters allowed for the customer name.

# I used json to store data because it is more organized.
# This function loads the hire records from the hired_items.json file.
# An empty list will be returned if there is no file or the file is empty.
def load_data():
    """Load hired records from the hired_items.json file or return an empty list if the file doesn't exist or is empty"""
    if os.path.exists("hired_items.json"): # Check if the file exists.
        try:
            with open("hired_items.json","r") as file:
                content=file.read()
                if content.strip()=="": # Check if the content in the file is empty.
                    return[]
                try:
                    return json.loads(content)
                except json.JSONDecodeError:
                    messagebox.showerror("Error","Data file is corrupted. Starting with empty records")
                    return[]
        except IOError as e:
            messagebox.showerror("Error",f"Could not load data: {e}")
            return[]
    return[]

# This function saves the updated hire records to the json file.
# This overwrites the file with the updated old and new records.
def save_data(data):
    """Save the given list of hire records to the hired_items.json file"""
    try:
        with open("hired_items.json","w") as file:
            json.dump(data,file,indent=4)
        return True # Return True so the program knows the records were saved.
    except Exception as e:
        messagebox.showerror("Error",f"Could not save data: {e}")
        return False # Return False so the program does not continue after a save error.

# I used this function to auto generate a random unique 6 digit receipt number.
def generate_receipt_no():
    """Generate a random 6 digit receipt number that is not already used"""
    data=load_data()
    used_receipts=[]
    for record in data:
        used_receipts.append(record["recei_no"])

    recei_no=random.randint(100000,999999) # Generate a random receipt number with 6 digits.
    while recei_no in used_receipts: # If the number already exists generate another random number.
        recei_no=random.randint(100000,999999)
    return recei_no

# Created a sidebar with buttons to navigate into home, add hire window and current hire window which make it easier for users to use.
# Active page is coloured navy blue so the user knows where he or she is(EX: If the user is in current hire page, the current hire button on sidebar is coloured navy blue).
def create_bar(home,hire_command=""):
    sidebar = tk.Frame(home,bg=TEAL,width=100,height=400,relief='flat')
    sidebar.pack(side='left',fill="y",expand=False)
    sidebar.pack_propagate(False)

    # Check if the logo file is in the same folder before trying to open it.
    # If the logo is not there the program will still run without showing the logo.
    if os.path.exists("Logo.png"):
        current_image=Image.open("Logo.png") # Adding the company logo to the sidebar.
        current_image=current_image.resize((80,50))# Resizing the image to fit in the sidebar.
        company_logo=ImageTk.PhotoImage(current_image)

        the_company_logo=tk.Label(sidebar,image=company_logo,bg=TEAL)
        the_company_logo.pack(side="top", pady=10)
        the_company_logo.image=company_logo

    # The "lambda" in "command=lambda:home_window() is used to create a small function in one line without needing a full def function".
    # Mostly here used for for buttons to run when a command is given.
    bar_btn_home=tk.Button(sidebar, text="Home", bg=TEAL if hire_command !="home" else NAVY_BLUE, fg=WHITE, relief='flat',command=lambda:home_window())
    bar_btn_home.pack(fill="x",padx=10, pady=5)

    bar_btn_add_hire=tk.Button(sidebar, text="Add Hire", bg=TEAL if hire_command !="add_hire" else NAVY_BLUE, fg=WHITE,bd=0, relief='flat',command=lambda:add_hire_window())
    bar_btn_add_hire.pack(fill="x",padx=10, pady=5)

    bar_btn_view_hires=tk.Button(sidebar, text="View Hires", bg=TEAL if hire_command !="view_hires" else NAVY_BLUE, fg=WHITE,bd=0, relief='flat',command=lambda:view_current_hires_window())
    bar_btn_view_hires.pack(fill="x",padx=10, pady=5)

    return sidebar
current_window=[None] # Keep track of the window that is currently opened and helps the program to close the old window.
current_page=[""] # This keeps track of the page that is currently opened and avoid opening the same page multiple times.

# This function closes the current window and resets the program.
# So the user can go back to the home page.
def home_window():
    if current_window[0] is not None:
        current_window[0].destroy()
        current_window[0] = None
    current_page[0]= ""
    root.deiconify()


# This function opens a new window to select an item to hire.
# This makes it easier for the user to select an item because each item has an icon.
# Using buttons means the user does not have to type the accessory name.
def selectitem_window(select_itm,sele_item_window_btn):
    select_item_popup=tk.Toplevel()
    select_item_popup.title("Accessory Selection Window")
    select_item_popup.geometry("600x400")
    select_item_popup.configure(bg=BG_CL)
    select_item_popup.grab_set()
    select_item_popup.resizable(False,False)

    temp_rory_sele=tk.StringVar(value=select_itm.get())

    sele_item_lbl=tk.Label(select_item_popup,text="Select an item",bg=BG_CL,font=("Times New Roman",20))
    sele_item_lbl.pack(pady=20)

    button_frame=tk.Frame(select_item_popup,bg=BG_CL)
    button_frame.pack(pady=10,padx=20)

    buttons=[]

    # Enumerate gives the number of the item and the item.
    # I used this to place the accessories in a grid.
    for i,item in enumerate(ACCESSORIES):
        row=i//2 # Assigned the row position for the buttons.
        col=i%2  # Assigned the column position for the buttons.
        btnbtn=tk.Button(button_frame,text=ACCESSORIES_ICONS[i],bg=TEAL,fg=WHITE,relief="flat",width=20,height=2)
        btnbtn.grid(row=row,column=col,padx=8,pady=6)
        buttons.append(btnbtn)
    # When an item is clicked this function runs.
    # This function temporarily stores the item that is selected.
    def click_itm(item):
        cln_item=ACCESSORIES[ACCESSORIES_ICONS.index(item)]
        temp_rory_sele.set(cln_item) # This stores the item that is selected.
        for btn in buttons:
            btn.config(bg=TEAL) # All the other button colours are reset back to their original colour.
        buttons[ACCESSORIES_ICONS.index(item)].config(bg=NAVY_BLUE) # Highlight the selected item button navy blue.

    # Here the selected button is linked to the click itm function.
    for btnbtn in buttons:
        btnbtn.config(command=lambda btn=btnbtn: click_itm(btn["text"]))

    if temp_rory_sele.get() in ACCESSORIES:
        buttons[ACCESSORIES.index(temp_rory_sele.get())].config(bg=NAVY_BLUE)
    # This function is used to confirm the selected item.
    # It replaces the "Select an item" button in the add hire window with the item that is selected.
    # Close the accessory selection window and go back to the add hire window.
    def confirm_sele():
        if temp_rory_sele.get() == "":
            return
        select_itm.set(temp_rory_sele.get())
        icn_txt=ACCESSORIES_ICONS[ACCESSORIES.index(temp_rory_sele.get())]
        sele_item_window_btn.config(text=icn_txt)
        select_item_popup.destroy()

    # This function closes the accessory selection window without saving the selected item.
    def cancel_sele():
        select_item_popup.destroy()

    btn_frame=tk.Frame(select_item_popup, bg=BG_CL)
    btn_frame.pack(pady=15)

    confirm_btn=tk.Button(btn_frame,text="Confirm Selection",bg=WHITE,fg=BLACK,relief="flat",width=20,command=confirm_sele)
    confirm_btn.grid(row=0,column=0,padx=10)

    cancel_btn=tk.Button(btn_frame,text="Cancel",bg=WHITE,fg=BLACK,relief="flat",width=20,command=cancel_sele)
    cancel_btn.grid(row=0,column=1,padx=10)

# This function shows a receipt with all the details entered by the user in a new pop-up window.
# It takes the new hire record as a parameter and displays all the hire details.
def receipt_shown(record):
    receipt_popup=tk.Toplevel()
    receipt_popup.title("Receipt")
    receipt_popup.geometry("350x360")
    receipt_popup.configure(bg=TEAL)
    receipt_popup.grab_set()
    receipt_popup.resizable(False,False)

    recei_frame=tk.Frame(receipt_popup,bg=BG_CL)
    recei_frame.pack(pady=5,fill="both",expand=True)
    recei_frame.columnconfigure(0,weight=1)
    recei_frame.columnconfigure(1,weight=1)

    receipt_lbl=tk.Label(recei_frame,text="Receipt",bg=BG_CL,font=("Times New Roman",20,"italic"))
    receipt_lbl.grid(row=0,column=0,columnspan=2,pady=3,sticky="ew")

    company_lbl=tk.Label(recei_frame,text="Byte & Bolt Tech Hire ",bg=BG_CL,font=("Times New Roman",20))
    company_lbl.grid(row=1,column=0,columnspan=2,pady=3,sticky="ew")

    separator_one=ttk.Separator(recei_frame, orient="horizontal") # This is used to seperate the upper part from the lower part to make it look good.
    separator_one.grid(row=2,column=0,columnspan=2,sticky="ew",pady=5)

    recei_no_lbl=tk.Label(recei_frame,text="Receipt number: ",bg=BG_CL)
    recei_no_lbl.grid(row=3,column=0,pady=5,padx=20,sticky="e")

    recei_lbl=tk.Label(recei_frame,text=str(record["recei_no"]),bg=BG_CL)
    recei_lbl.grid(row=3,column=1,pady=5,padx=20,sticky="w")

    recei_name_lbl=tk.Label(recei_frame,text="Customer name: ",bg=BG_CL)
    recei_name_lbl.grid(row=4,column=0,pady=5,padx=20,sticky="e")

    name_lbl=tk.Label(recei_frame,text=str(record["cus_name"]),bg=BG_CL)
    name_lbl.grid(row=4,column=1,pady=5,padx=20,sticky="w")

    recei_item_lbl=tk.Label(recei_frame,text="Hired Item:  ",bg=BG_CL)
    recei_item_lbl.grid(row=5,column=0,pady=5,padx=20,sticky="e")

    item_lbl=tk.Label(recei_frame,text=str(record["selected_item"]),bg=BG_CL)
    item_lbl.grid(row=5,column=1,pady=5,padx=20,sticky="w")

    recei_qty_lbl=tk.Label(recei_frame,text="Quantity hired: ",bg=BG_CL)
    recei_qty_lbl.grid(row=6,column=0,pady=5,padx=20,sticky="e")

    qty_lbl=tk.Label(recei_frame,text=str(record["quantity_hired"]),bg=BG_CL)
    qty_lbl.grid(row=6,column=1,pady=5,padx=20,sticky="w")

    recei_date_lbl=tk.Label(recei_frame,text="Date hired: ",bg=BG_CL)
    recei_date_lbl.grid(row=7,column=0,pady=5,padx=20,sticky="e")

    date_lbl=tk.Label(recei_frame,text=str(record["date_selected"]),bg=BG_CL)
    date_lbl.grid(row=7,column=1,pady=5,padx=20,sticky="w")

    separator_two=ttk.Separator(recei_frame, orient="horizontal")
    separator_two.grid(row=8,column=0,columnspan=2,sticky="ew",pady=5)

    thank_label=tk.Label(recei_frame,text="Thanks for coming!",bg=BG_CL,font=("arial",20))
    thank_label.grid(row=9,column=0,columnspan=2,pady=5)

    # When the Home button is pressed it closes the receipt and returns to the home window.
    recei_home_btn=tk.Button(receipt_popup,text="Home",command=lambda:[receipt_popup.destroy(),home_window()],bg=TEAL,fg=WHITE,width=20,height=3,relief="flat")
    recei_home_btn.pack(pady=5)

# Opens the add hire window where the staff can enter the hire details.
def add_hire_window():
    if current_page[0]=="add_hire": # If the current page is add hire window it keeps it and avoid opening it multiple times when the add hire button is clicked on sidebar or the home screen.
        return
    if current_window[0] is not None:
        current_window[0].destroy()
    root.withdraw() # Here it hides the home window when the add hire window is opened.

    addh=tk.Toplevel(root)
    addh.title("Add Hire Record")
    addh.geometry("600x400")
    addh.resizable(False,False)
    current_window[0]= addh
    current_page[0]="add_hire"

    def close():
        current_window[0]= None
        current_page[0]= ""
        root.deiconify()
        addh.destroy()

    addh.protocol("WM_DELETE_WINDOW", close)

    create_bar(addh,hire_command="add_hire") # Add the sidebar to the home screen.

    addh_content=tk.Frame(addh,bg=BG_CL)
    addh_content.pack(side="right" , fill= "both" , expand=True)
    addh_content.columnconfigure(0,weight=1)
    addh_content.columnconfigure(1,weight=2)

    # This function opens a pop up calendar window so the user can select a date.
    def get_cal_date():
        cale_popup=tk.Toplevel()
        cale_popup.title("Select Date")
        cale_popup.geometry("300x300")
        cale_popup.grab_set()
        cale_popup.resizable(False,False)

        calendar=Calendar(cale_popup,selectmode="day",date_pattern="mm/dd/yyyy",
                          year=date.today().year,
                          month=date.today().month,
                          day=date.today().day,
                          mindate=date(2000,1,1))
        calendar.grid(row=0,column=0,padx=10,pady=10)

        # This function confirms the date selected from the calendar.
        # It replaces the date entry on the Add Hire window with the selected date.
        def date_confirm():
            date_sele_variable.set(calendar.get_date())
            cale_popup.destroy()

        confirm_btn=tk.Button(cale_popup,text="Confirm Date",command=date_confirm)
        confirm_btn.grid(row=1,column=0,pady=5)
    # This function is used to validate the entries entered by the users.
    # If the entered data is valid it saves the record into the JSON file and opens a receipt window.
    def selection_confirm():
        """Validate the entered hired details and if valid save the record and show a receipt"""
        # Here it collects all form field values for validation.
        customer_name=cus_name_entry.get().strip()
        selected_item=select_itm.get()
        quantity_hired=qty_sele_spin.get()
        date_selected=date_entry.get()

        # Check all the validations and collect any errors that occurred.
        errors=[]
        # Check if the customer name is empty or only contains letters.
        # If the name is invalid, add an error message to the list.
        if not customer_name or not customer_name.replace(" ","").isalpha():
            errors.append("Customer name cannot be empty or contain integers or any special characters.")
        if len(customer_name)>NAME_MAX:
            errors.append("Customer name should be 15 characters or fewer.")

        # Check if an item is selected and add an error message if it is empty.
        if selected_item =="":
            errors.append("Please select an accessory item from the item selection window.")

        # Check that the quantity is a whole number from 1 to 20.
        if quantity_hired =="" or not quantity_hired.isdigit() or int(quantity_hired) < QTY_MIN or int(quantity_hired) > QTY_MAX:
            errors.append("Quantity must be a whole number between 1 and 20.")

        # Check if a date is selected.
        if date_selected =="":
            errors.append("Please select a hire date from the calendar.")
        # Check if the selected date is in the past.
        # If it is in the past, add an error message.
        else:
            try:
                chosen_date=datetime.strptime(date_selected,"%m/%d/%Y").date()
                if chosen_date.year < 2000:
                    errors.append("Date cannot be before the year 2000.")
                elif chosen_date < date.today():
                    errors.append("Date cannot be in the past. Please select today or a future date.")
            except ValueError:
                errors.append("Please select a valid hire date from the calendar.")

        # If there are any errors, join them and show them all at once.
        if errors:
            messagebox.showerror("Error","\n".join(errors))
            return
        # Create a new hire record and then save it to the JSON file.
        new_record={
            "recei_no":int(recei_variable.get()),
            "cus_name":customer_name,
            "selected_item":selected_item,
            "quantity_hired":int(quantity_hired),
            "date_selected":date_selected
        }

        records=load_data()
        records.append(new_record)
        saved=save_data(records)
        if not saved:
            return

        # Generate a new random receipt number for the next hire before showing the receipt.
        # The other entered fields are kept the same and only the receipt number is changed.
        recei_variable.set(str(generate_receipt_no()))
        receipt_shown(new_record)

    # This function clears all the entered details and resets the form fields.
    def clear_form():
        cus_name_variable.set("")
        select_itm.set("")
        sele_item_window_btn.config(text="Select Item")
        qty_sele_spin.set("")
        date_sele_variable.set("")


    addh_label=tk.Label(addh_content, text = "Add Hire Record", font=("Times New Roman",25,"underline") , bg=BG_CL)
    addh_label.grid(row=0,column=0,columnspan=2,sticky="w",pady=15)

    # Auto generate a receipt number for the hire.
    recei_variable = tk.StringVar(value=str(generate_receipt_no()))

    recei_label=tk.Label(addh_content,text="Receipt Number:",bg=BG_CL,font=FONT_LBL,width=15)
    recei_label.grid(row=1,column=0, sticky="e", padx=14,pady=10)

    recei_entry=tk.Entry(addh_content,textvariable=recei_variable, state="readonly",fg=BLACK,width=25)
    recei_entry.grid(row=1,column=1,sticky="e",padx=20,pady=10)

    cus_name_variable=tk.StringVar()
    cus_name_label=tk.Label(addh_content,text="Customer name:",bg=BG_CL,font=FONT_LBL,width=15)
    cus_name_label.grid(row=2,column=0, sticky="e", padx=15,pady=10)

    cus_name_entry=tk.Entry(addh_content,textvariable=cus_name_variable,width=25)
    cus_name_entry.grid(row=2,column=1,sticky="e", padx=20,pady=10)

    select_itm=tk.StringVar(value="")
    sele_item_label=tk.Label(addh_content,text="Select Item:       ",bg=BG_CL,font=FONT_LBL,width=15)
    sele_item_label.grid(row=3,column=0, sticky="e", padx=15,pady=10)

    sele_item_window_btn=tk.Button(addh_content,text="Select Item",bg=TEAL,fg=WHITE,relief="flat",width=20)
    sele_item_window_btn.grid(row=3,column=1, sticky="e", padx=20,pady=10)
    sele_item_window_btn.config(command=lambda:selectitem_window(select_itm,sele_item_window_btn))

    qty_sele_label=tk.Label(addh_content,text="Quantity Hired: ",bg=BG_CL,font=FONT_LBL,width=15)
    qty_sele_label.grid(row=4,column=0,sticky="e",padx=15,pady=10)

    qty_sele_spin=ttk.Spinbox(addh_content,from_=QTY_MIN,to=QTY_MAX,width=22,state="readonly")
    qty_sele_spin.grid(row=4,column=1,sticky="e", padx=20,pady=10)

    date_sele_variable=tk.StringVar()
    date_sele_lbl=tk.Label(addh_content,text="Date Hired:      ",bg=BG_CL,font=FONT_LBL,width=15)
    date_sele_lbl.grid(row=5,column=0,sticky="e",padx=15,pady=10)

    date_entry=tk.Entry(addh_content,textvariable=date_sele_variable, state="readonly",fg=BLACK,width=25)
    date_entry.grid(row=5,column=1,sticky="e",padx=20,pady=10)

    cal_btn=tk.Button(addh_content, text="📆", bg=TEAL, fg=WHITE,bd=0, relief='flat',width=3, height=2,command=get_cal_date)
    cal_btn.grid(row=5,column=2,sticky="e",padx=8,pady=10)

    add_hire_btn_frame=tk.Frame(addh_content,bg=BG_CL)
    add_hire_btn_frame.grid(row=6,column=0,columnspan=2,pady=10)

    confirm_sele=tk.Button(add_hire_btn_frame, text="Confirm Selection", bg=TEAL ,fg=WHITE,relief='flat',width=15, height=2,command=selection_confirm)
    confirm_sele.pack(side="left", padx=10)

    clear_selection_btn=tk.Button(add_hire_btn_frame,text="Clear",bg=TEAL,fg=WHITE,relief="flat",width=15,height=2,command=clear_form)
    clear_selection_btn.pack(side="left",padx=10)

# This function opens the view hire window where the staff can delete records when the item is returned.
# Shows all the hires in a table.
def view_current_hires_window():
    if current_page[0]=="view_hires":
        return
    if current_window[0] is not None:
        current_window[0].destroy()
    root.withdraw()

    viewh=tk.Toplevel(root)
    viewh.title("Current Hires")
    viewh.geometry("620x400")
    viewh.resizable(False,False)
    current_window[0]= viewh
    current_page[0]="view_hires"
    # When the window is closed it automatically opens the home window.
    def close():
        current_window[0]= None
        current_page[0]= ""
        root.deiconify()
        viewh.destroy()

    viewh.protocol("WM_DELETE_WINDOW", close)

    # This function clears the current table and adds the records given to it.
    # I used one function for this so I do not have to repeat the same table code in search, clear and delete.
    def refresh_table(records):
        for row in table.get_children():
            table.delete(row)
        for r in records:
            table.insert("","end",values=(r["recei_no"],r["cus_name"],r["selected_item"],r["quantity_hired"],r["date_selected"]))

    # Delete the selected hire record from the JSON file and remake the table.
    def delete_record():
        try:
            selected_item=table.selection()
            if not selected_item:
                messagebox.showerror("Error","Please select a record to delete")
                return

            # Get all the values from the exact row that the user selected.
            selected_row=selected_item[0]
            selected_values=table.item(selected_row)["values"]

            # Find only one matching record and remove that record from the json file.
            records=load_data()
            record_found=False
            for i,r in enumerate(records):
                if (str(r["recei_no"])==str(selected_values[0]) and
                    str(r["cus_name"])==str(selected_values[1]) and
                    str(r["selected_item"])==str(selected_values[2]) and
                    str(r["quantity_hired"])==str(selected_values[3]) and
                    str(r["date_selected"])==str(selected_values[4])):
                    del records[i]
                    record_found=True
                    break

            if not record_found:
                messagebox.showerror("Error","The selected record could not be found")
                return

            saved=save_data(records)
            if not saved:
                return

            # Remake the table using the records that are still saved.
            refresh_table(load_data())
            messagebox.showinfo("Success","Record deleted successfully")

        # If there are any unexpected errors, show an error message.
        except Exception as f:
            messagebox.showerror("Error",f"Can't delete record:{f}")

    # Add the sidebar to the current hire window.
    create_bar(viewh,hire_command="view_hires")

    # This function is used to search a record in the table by its name and show all the records that are similar to the searched name.
    def records_search():
        search_word=search_variable.get().strip().lower()
        if search_word=="":
            messagebox.showerror("Error","Please enter a customer name to search.")
            return
        if not search_word.replace(" ","").isalpha():
            messagebox.showerror("Error","Search name can only contain letters.")
            return

        matching_records=[]
        for r in load_data():
            if search_word in r["cus_name"].lower():
                matching_records.append(r)
        refresh_table(matching_records)

    # This function clears the search entry and displays all the records again.
    def clr_search():
        search_variable.set("")
        refresh_table(load_data())


    viewh_content=tk.Frame(viewh,bg=BG_CL)
    viewh_content.pack(side="right" , fill= "both" , expand=True)

    viewh_label=tk.Label(viewh_content, text = "Current Hires", font=("Times New Roman",25,"underline") , bg=BG_CL,anchor="w")
    viewh_label.pack(pady=20,fill="x")

    search_frm=tk.Frame(viewh_content,bg=BG_CL)
    search_frm.pack(pady=5)

    search_variable=tk.StringVar()
    view_search=tk.Entry(search_frm,textvariable=search_variable,width=30)
    view_search.grid(row=0,column=0,padx=5)

    search_btn=tk.Button(search_frm,text="Search",command=records_search,bg=TEAL,fg=WHITE, relief="flat")
    search_btn.grid(row=0,column=1,padx=5)

    clear_search_btn=tk.Button(search_frm,text="Clear",command=clr_search,width=30)
    clear_search_btn.grid(row=0,column=2,padx=5)

    # Makes a table with columns and rows with all the records in each column.
    columns=["receipt_no","custo_name","hired_item","quantity_hired","date_hired"]
    table=ttk.Treeview(viewh_content,columns=columns, show="headings",height=8)

    table.heading("receipt_no",text="Receipt No.")
    table.column("receipt_no",width=100,anchor="center")


    table.heading("custo_name",text="Customer name")
    table.column("custo_name",width=100,anchor="center")


    table.heading("hired_item",text="Accessory Hired")
    table.column("hired_item",width=100,anchor="center")


    table.heading("quantity_hired",text="Quantity Hired")
    table.column("quantity_hired",width=100,anchor="center")


    table.heading("date_hired",text="Date Hired")
    table.column("date_hired",width=100,anchor="center")
    table.pack(pady=10)

    # Insert all the saved records into the table using the same refresh function.
    refresh_table(load_data())

    current_hire_delerecord_btn=tk.Button(viewh_content,text="Delete Record",command=delete_record,bg=TEAL,fg=WHITE,width=20,height=2,relief="flat")
    current_hire_delerecord_btn.pack(side="left",pady=5,padx=40)

    current_hire_home_btn=tk.Button(viewh_content,text="Home",command=close,bg=TEAL,fg=WHITE,width=20,height=2,relief="flat")
    current_hire_home_btn.pack(side="right",pady=5,padx=40)

# Main function that builds and launches the home window of the program.
def main():
    global root # Make the root accessible for all the functions in the program.
    root = tk.Tk()
    root.title("Byte and Bolt Tech Hire")
    root.geometry("600x400")
    root.resizable(False,False)

    # Creates the sidebar to the home screen.
    create_bar(root,hire_command="home")

    # Main content of the home window.
    content=tk.Frame(root, bg=BG_CL)
    content.pack(side="right" , fill= "both" , expand=True)

    topic_label_one=tk.Label(content, text="Welcome to" , font=("Times New Roman",30,"italic") , bg=BG_CL)
    topic_label_one.pack(pady=5)

    topic_label_two=tk.Label(content, text="Byte and Bolt Tech Hire" , font=("Comic Sans MS",25,"bold") , bg=BG_CL)
    topic_label_two.pack(pady=5)

    # Main buttons to navigate into each window.
    main_addhire_btn=tk.Button(content, text="Add Hire record", bg=NAVY_BLUE, fg=WHITE,bd=0, relief='flat',width=30, height=2,command=add_hire_window)
    main_addhire_btn.pack(padx=10, pady=5)

    main_viewhire_btn=tk.Button(content, text="View Current Hires", bg=TEAL ,fg=WHITE,bd=0, relief='flat',width=30, height=2,command=view_current_hires_window)
    main_viewhire_btn.pack(padx=10, pady=5)

    main_exit_btn=tk.Button(content, text="Exit", bg=LIGHT_TEAL , fg=WHITE,bd=0, relief='flat',width=30, height=2,command=root.destroy)
    main_exit_btn.pack(padx=10, pady=5)

    root.mainloop() # Start the tkinter loop to keep open the windows.

# Run the program if the file is run directly.
if __name__ =="__main__":
    main()

