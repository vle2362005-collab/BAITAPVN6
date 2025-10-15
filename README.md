using EntityFramework5.Model;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace EntityFramework5
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            try
            {
                StudentContextDB context = new StudentContextDB();
                List<Faculty> listFalcultys = context.Faculties.ToList(); //lấy các khoa
                List<Student> listStudent = context.Students.ToList(); //lấy sinh viên
                FillFalcultyCombobox(listFalcultys);
                BindGrid(listStudent);
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
        private void FillFalcultyCombobox(List<Faculty> listFalcultys)
        {
            this.cmbFaculty.DataSource = listFalcultys;
            this.cmbFaculty.DisplayMember = "FacultyName";
            this.cmbFaculty.ValueMember = "FacultyID";
        }
        private void BindGrid(List<Student> listStudent)
        {
            // Đảm bảo DataGridView đã có cột trước khi thêm dòng
            if (dgvStudent.Columns.Count == 0)
            {
                dgvStudent.Columns.Add("StudentID", "Mã SV");
                dgvStudent.Columns.Add("FullName", "Họ tên");
                dgvStudent.Columns.Add("FacultyName", "Khoa");
                dgvStudent.Columns.Add("AverageScore", "Điểm TB");
            }

            // Đảm bảo DataGridView đã có cột trước khi thêm dòng
            if (dgvStudent.Columns.Count == 0)
            {
                dgvStudent.Columns.Add("StudentID", "Mã SV");
                dgvStudent.Columns.Add("FullName", "Họ tên");
                dgvStudent.Columns.Add("FacultyName", "Khoa");
                dgvStudent.Columns.Add("AverageScore", "Điểm TB");
            }

                dgvStudent.Rows.Clear();
            foreach (var item in listStudent)
            {
                int index = dgvStudent.Rows.Add();
                dgvStudent.Rows[index].Cells[0].Value = item.StudentID;
                dgvStudent.Rows[index].Cells[1].Value = item.FullName;
                dgvStudent.Rows[index].Cells[2].Value = item.Faculty.FacultyName;
                dgvStudent.Rows[index].Cells[3].Value = item.AverageScore;
            }
        }

        private void dataGridView1_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {
            if (e.RowIndex >= 0)
            {
                DataGridViewRow selectedRow = dgvStudent.Rows[e.RowIndex];
                txtStudentId.Text = selectedRow.Cells[0].Value.ToString();
                txtFullName.Text = selectedRow.Cells[1].Value.ToString();
                cmbFaculty.Text = selectedRow.Cells[2].Value.ToString();
                txtAverageScore.Text = selectedRow.Cells[3].Value.ToString();
            }
        }

        private void btnAdd_Click_Click(object sender, EventArgs e)
        {
            try
            {
                StudentContextDB context = new StudentContextDB();
                List<Student> studentList = context.Students.ToList();
                if (studentList.Any(s => s.StudentID == txtStudentId.Text))
                {
                    MessageBox.Show("Mã SV đã tồn tại. Vui lòng nhập một mã khác.", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                    return;
                }

                var newStudent = new Student
                {
                    StudentID = txtStudentId.Text,
                    FullName = txtFullName.Text,
                    AverageScore = double.Parse(txtAverageScore.Text),
                    FacultyID = cmbFaculty.SelectedValue.ToString(),
                };

                // Add the new student to the list
                context.Students.Add(newStudent);
                context.SaveChanges();

                // Reload the data
                context.Students.ToList();
                MessageBox.Show("Thêm sinh viên thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Lỗi khi thêm dữ liệu: {ex.Message}", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        // Add this method to handle the TextChanged event for the text boxes.
        // This will resolve CS1061 by providing the missing event handler.
        private void textBox1_TextChanged(object sender, EventArgs e)
        {
            // You can leave this empty or add logic as needed.
        }

        private void btnUpdate_Click_Click(object sender, EventArgs e)
        {
            try
            {
                StudentContextDB db = new StudentContextDB();
                List<Student> studentList = db.Students.ToList();

                var student = studentList.FirstOrDefault(s => s.StudentID == txtStudentId.Text);
                if (student != null)
                {
                    if (studentList.Any(s => s.StudentID == txtStudentId.Text && s.StudentID != student.StudentID))
                    {
                        MessageBox.Show("Mã SV đã tồn tại. Vui lòng nhập một mã khác.", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
                        return;
                    }
                    student.FullName = txtFullName.Text;
                    student.AverageScore = double.Parse(txtAverageScore.Text);
                    student.FacultyID = cmbFaculty.SelectedValue.ToString();

                    // Save changes to the database
                    db.SaveChanges();

                    // Reload the data
                    db.Students.ToList();
                    MessageBox.Show("Chỉnh sửa thông tin sinh viên thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
                else
                {
                    MessageBox.Show("Sinh viên không tìm thấy!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Lỗi khi cập nhật dữ liệu: {ex.Message}", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        private void btnDelete_Click_Click(object sender, EventArgs e)
        {
            try
            {
                StudentContextDB context = new StudentContextDB();
                List<Student> studentList = context.Students.ToList();

                // Find the student by Student ID
                var student = studentList.FirstOrDefault(s => s.StudentID == txtStudentId.Text);
                if (student != null)
                {
                    // Remove the student from the list
                    context.Students.Remove(student);
                    context.SaveChanges();

                    context.Students.ToList();
                    MessageBox.Show("Sinh viên đã được xoá thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
                else
                {
                    MessageBox.Show("Sinh viên không tìm thấy!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Lỗi khi cập nhật dữ liệu: {ex.Message}", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        private void btnOut_Click_Click(object sender, EventArgs e)
        {
            this.Close();
        }
    }
}
