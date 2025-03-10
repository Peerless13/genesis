<script lang="ts">
    import { onMount } from 'svelte';
  
    type Student = {
      name: string;
      grades: number[];
    };
  
    type Assignment = {
      name: string;
      weight: number;
      grade: number;
    };
  
    let students: Student[] = [];
    let assignments: Assignment[] = [];
    let newStudentName = '';
    let newAssignmentName = '';
    let newAssignmentWeight = 0;
    let newAssignmentGrade = 0;
  
    let isSidebarOpen = true;
  
    onMount(() => {
      try {
        const storedStudents = localStorage.getItem('students');
        const storedAssignments = localStorage.getItem('assignments');
        students = storedStudents ? JSON.parse(storedStudents) : [];
        assignments = storedAssignments ? JSON.parse(storedAssignments) : [];
      } catch (error) {
        console.error('Error accessing localStorage:', error);
      }
    });
  
    function saveData(): void {
      try {
        localStorage.setItem('students', JSON.stringify(students));
        localStorage.setItem('assignments', JSON.stringify(assignments));
      } catch (error) {
        console.error('Error saving to localStorage:', error);
      }
    }
  
    function addStudent(): void {
      if (newStudentName.trim() !== '') {
        students.push({
          name: newStudentName,
          grades: []
        });
        newStudentName = '';
        saveData();
      }
    }
  
    function addAssignment(): void {
      if (newAssignmentName.trim() !== '' && newAssignmentWeight > 0) {
        assignments.push({
          name: newAssignmentName,
          weight: newAssignmentWeight,
          grade: newAssignmentGrade
        });
        newAssignmentName = '';
        newAssignmentWeight = 0;
        newAssignmentGrade = 0;
        saveData();
      }
    }
  
    function updateGrade(student: Student, assignment: Assignment, newGrade: number): void {
      const assignmentIndex = assignments.indexOf(assignment);
      if (assignmentIndex !== -1) {
        assignments[assignmentIndex].grade = newGrade;
        const studentIndex = students.indexOf(student);
        if (studentIndex !== -1) {
          const studentGrades = students[studentIndex].grades;
          const assignmentIndexInGrades = studentGrades.indexOf(assignment.grade);
          if (assignmentIndexInGrades !== -1) {
            studentGrades[assignmentIndexInGrades] = newGrade;
          }
        }
        saveData();
      }
    }
  
    function calculateGrade(student: Student): string {
      let totalWeight = 0;
      let weightedGrade = 0;
      assignments.forEach((assignment: Assignment) => {
        weightedGrade += assignment.grade * (assignment.weight / 100);
        totalWeight += assignment.weight;
      });
      const gradePercentage = weightedGrade * 100 / totalWeight;
      return getLetterGrade(gradePercentage);
    }
  
    function getLetterGrade(percentage: number): string {
      if (percentage >= 90) return 'A';
      if (percentage >= 80) return 'B';
      if (percentage >= 70) return 'C';
      if (percentage >= 60) return 'D';
      return 'F';
    }
  
    // Toggle sidebar visibility
    function toggleSidebar(): void {
      isSidebarOpen = !isSidebarOpen;
    }
  </script>
  
  <main class="container">
    <div class="sidebar" class:closed={!isSidebarOpen}>
      <h2>Genesis System</h2>
      <nav>
        <ul>
          <li><a href="#students">Students</a></li>
          <li><a href="#assignments">Assignments</a></li>
          <li><a href="#settings">Settings</a></li>
        </ul>
      </nav>
      <button on:click={toggleSidebar} class="toggle-button">&#9776;</button>
    </div>
  
    <div class="content">
      <header>
        <h1>Student Grading System</h1>
        <p>Manage students and assignments efficiently.</p>
      </header>
  
      <section id="students" class="card">
        <h2>Students</h2>
        <input type="text" bind:value={newStudentName} placeholder="Enter student name" class="input" />
        <button on:click={addStudent} class="button">Add Student</button>
  
        <ul class="list">
          {#each students as student (student.name)}
            <li class="list-item">
              <div class="student-info">
                <div class="student-name">
                  {student.name} - Grade: {calculateGrade(student)}
                </div>
                <div class="student-assignments">
                  <ul>
                    {#each assignments as assignment}
                      <li>
                        <div class="assignment-info">
                          {assignment.name} - 
                          <input type="number" bind:value={assignment.grade} min="0" max="100" class="input" 
                            on:blur={() => updateGrade(student, assignment, assignment.grade)} 
                            placeholder="Edit Grade" />
                        </div>
                      </li>
                    {/each}
                  </ul>
                </div>
              </div>
            </li>
          {/each}
        </ul>
      </section>
  
      <section id="assignments" class="card">
        <h2>Assignments</h2>
        <input type="text" bind:value={newAssignmentName} placeholder="Enter assignment name" class="input" />
        <input type="number" bind:value={newAssignmentWeight} placeholder="Weight" min="1" class="input" />
        <input type="number" bind:value={newAssignmentGrade} placeholder="Grade" min="0" max="100" class="input" />
        <button on:click={addAssignment} class="button">Add Assignment</button>
  
        <ul class="list">
          {#each assignments as assignment (assignment.name)}
            <li class="list-item">
              {assignment.name} - Weight: {assignment.weight}% - Grade: {assignment.grade}%
            </li>
          {/each}
        </ul>
      </section>
  
      <footer>
        <button on:click={saveData} class="button save">Save Data</button>
      </footer>
    </div>
  </main>
  
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #6a11cb, #2575fc);
      color: white;
    }
  
    .container {
      display: flex;
      min-height: 100vh;
    }
  
    .sidebar {
      width: 250px;
      background-color: #2d3a3f;
      padding: 20px;
      color: white;
      position: fixed;
      height: 100vh;
      transition: transform 0.3s ease;
    }
  
    .sidebar.closed {
      transform: translateX(-100%);
    }
  
    .sidebar h2 {
      font-size: 1.5rem;
      margin-bottom: 30px;
      color: #fff;
    }
  
    .sidebar nav ul {
      list-style-type: none;
      padding: 0;
    }
  
    .sidebar nav ul li {
      margin: 15px 0;
    }
  
    .sidebar nav ul li a {
      text-decoration: none;
      color: #b0b0b0;
      font-size: 1.1rem;
    }
  
    .sidebar nav ul li a:hover {
      color: #ffffff;
    }
  
    .content {
      margin-left: 250px;
      padding: 20px;
      flex: 1;
      background-color: #f4f6f8;
      transition: margin-left 0.3s ease;
    }
  
    .content .closed {
      margin-left: 0;
    }
  
    .toggle-button {
      position: absolute;
      top: 20px;
      right: -40px;
      font-size: 30px;
      background: #333;
      color: white;
      border: none;
      padding: 10px;
      cursor: pointer;
      border-radius: 5px;
    }
  
    .header {
      text-align: center;
      margin-bottom: 30px;
    }
  
    header h1 {
      font-size: 2.5rem;
      margin-bottom: 10px;
    }
  
    header p {
      font-size: 1.2rem;
      color: #555;
    }
  
    .card {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
      padding: 20px;
      margin-bottom: 30px;
    }
  
    .input {
      padding: 10px;
      font-size: 1rem;
      border: 1px solid #ddd;
      border-radius: 5px;
      margin-right: 10px;
      width: 200px;
      margin-bottom: 15px;
    }
  
    .button {
      padding: 10px 20px;
      font-size: 1rem;
      border: none;
      border-radius: 5px;
      background-color: #4CAF50;
      color: white;
      cursor: pointer;
    }
  
    .button:hover {
      background-color: #45a049;
    }
  
    .list {
      list-style-type: none;
      padding: 0;
    }
  
    .list-item {
      padding: 10px;
      background: #f9f9f9;
      border-radius: 5px;
      margin-bottom: 10px;
    }
  
    .student-info {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
  
    .student-name {
      font-weight: bold;
    }
  
    .student-assignments ul {
      padding-left: 20px;
    }
  
    .assignment-info {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
  
    footer {
      text-align: center;
    }
  
    .save {
      background-color: #007BFF;
    }
  
    .save:hover {
      background-color: #0056b3;
    }
  </style>
  