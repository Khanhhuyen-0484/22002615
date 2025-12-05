types/student.ts 

export interface Student { 

  id: string; 

  name: string; 

  studentId: string; 

  class?: string; 

  email?: string; 

  phone?: string; 

} 

  

export interface StudentForm { 

  name: string; 

  studentId: string; 

  class?: string; 

  email?: string; 

  phone?: string; 

} 

  

export interface StudentState { 

  items: Student[]; 

  status: 'idle' | 'loading' | 'succeeded' | 'failed'; 

  error: string | null; 

} 

store/index.ts 

import { configureStore } from '@reduxjs/toolkit'; 

import studentReducer from './studentSlice'; 

  

export interface RootState { 

  students: { 

    items: Array<{ 

      id: string; 

      name: string; 

      studentId: string; 

      class?: string; 

      email?: string; 

      phone?: string; 

    }>; 

    status: 'idle' | 'loading' | 'succeeded' | 'failed'; 

    error: string | null; 

  }; 

} 

  

export const store = configureStore({ 

  reducer: { 

    students: studentReducer 

  } 

}); 

  

export type AppDispatch = typeof store.dispatch; 

store/studentSlice.ts 

import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'; 

import axios from 'axios'; 

  

const API_URL = 'https://6615b2d2b8b8e32ffc7a6ab9.mockapi.io/students'; 

  

export const fetchStudents = createAsyncThunk( 

  'students/fetchAll', 

  async () => { 

    const response = await axios.get(API_URL); 

    return response.data; 

  } 

); 

  

export const addStudent = createAsyncThunk( 

  'students/add', 

  async (studentData: any) => { 

    const response = await axios.post(API_URL, studentData); 

    return response.data; 

  } 

); 

  

export const updateStudent = createAsyncThunk( 

  'students/update', 

  async ({ id, ...studentData }: any) => { 

    const response = await axios.put(`${API_URL}/${id}`, studentData); 

    return response.data; 

  } 

); 

  

export const deleteStudent = createAsyncThunk( 

  'students/delete', 

  async (id: string) => { 

    await axios.delete(`${API_URL}/${id}`); 

    return id; 

  } 

); 

  

const studentSlice = createSlice({ 

  name: 'students', 

  initialState: { 

    items: [], 

    status: 'idle', 

    error: null 

  }, 

  reducers: {}, 

  extraReducers: (builder) => { 

    builder 

      .addCase(fetchStudents.pending, (state) => { 

        state.status = 'loading'; 

      }) 

      .addCase(fetchStudents.fulfilled, (state, action) => { 

        state.status = 'succeeded'; 

        state.items = action.payload; 

      }) 

      .addCase(fetchStudents.rejected, (state, action) => { 

        state.status = 'failed'; 

        state.error = action.error.message; 

      }) 

      .addCase(addStudent.fulfilled, (state, action) => { 

        state.items.push(action.payload); 

      }) 

      .addCase(updateStudent.fulfilled, (state, action) => { 

        const index = state.items.findIndex(s => s.id === action.payload.id); 

        if (index !== -1) { 

          state.items[index] = action.payload; 

        } 

      }) 

      .addCase(deleteStudent.fulfilled, (state, action) => { 

        state.items = state.items.filter(s => s.id !== action.payload); 

      }); 

  } 

}); 

  

export default studentSlice.reducer; 

app/_layout.tsx 

import { Stack } from 'expo-router'; 

import { Provider } from 'react-redux'; 

import { store } from '../store'; 

  

export default function RootLayout() { 

  return ( 

    <Provider store={store}> 

      <Stack> 

        <Stack.Screen name="index" options={{ title: 'Danh sách sinh viên' }} /> 

        <Stack.Screen name="add" options={{ title: 'Thêm sinh viên' }} /> 

        <Stack.Screen name="[id]" options={{ title: 'Chi tiết sinh viên' }} /> 

      </Stack> 

    </Provider> 

  ); 

} 

app/index.tsx 

import { useState, useEffect, useCallback, useMemo } from 'react'; 

import { View, FlatList } from 'react-native'; 

import { useSelector, useDispatch } from 'react-redux'; 

import { useRouter } from 'expo-router'; 

import { RootState } from '../store'; 

import { fetchStudents, deleteStudent } from '../store/studentSlice'; 

  

interface StudentItemProps { 

  item: any; 

  onEdit: (id: string) => void; 

  onDelete: (id: string) => void; 

} 

  

const StudentItem = ({ item, onEdit, onDelete }: StudentItemProps) => ( 

  <View style={{ padding: 10, borderBottomWidth: 1, borderColor: '#ccc' }}> 

    <View> 

      <View style={{ fontSize: 16, fontWeight: 'bold' }}>{item.name}</View> 

      <View>MSSV: {item.studentId}</View> 

      {item.class && <View>Lớp: {item.class}</View>} 

    </View> 

    <View style={{ flexDirection: 'row', marginTop: 5 }}> 

      <View onPress={() => onEdit(item.id)} style={{ marginRight: 10 }}> 

        <View style={{ color: 'blue' }}>Sửa</View> 

      </View> 

      <View onPress={() => onDelete(item.id)}> 

        <View style={{ color: 'red' }}>Xóa</View> 

      </View> 

    </View> 

  </View> 

); 

  

export default function Home() { 

  const router = useRouter(); 

  const dispatch = useDispatch(); 

   

  const { items, status } = useSelector((state: RootState) => state.students); 

  const [search, setSearch] = useState(''); 

  const [refreshing, setRefreshing] = useState(false); 

  

  useEffect(() => { 

    dispatch(fetchStudents() as any); 

  }, []); 

  

  const handleRefresh = useCallback(() => { 

    setRefreshing(true); 

    dispatch(fetchStudents() as any).finally(() => setRefreshing(false)); 

  }, [dispatch]); 

  

  const handleDelete = useCallback((id: string) => { 

    dispatch(deleteStudent(id) as any); 

  }, [dispatch]); 

  

  const filteredStudents = useMemo(() => { 

    if (!search.trim()) return items; 

    return items.filter(s => 

      s.name.toLowerCase().includes(search.toLowerCase()) || 

      s.studentId.toLowerCase().includes(search.toLowerCase()) 

    ); 

  }, [items, search]); 

  

  if (status === 'loading') { 

    return ( 

      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}> 

        <View>Đang tải...</View> 

      </View> 

    ); 

  } 

  

  return ( 

    <View style={{ flex: 1, padding: 10 }}> 

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={search} 

        onChangeText={setSearch} 

        placeholder="Tìm kiếm..." 

      /> 

  

      <View 

        onPress={() => router.push('/add')} 

        style={{ 

          backgroundColor: '#2196F3', 

          padding: 12, 

          borderRadius: 5, 

          alignItems: 'center', 

          marginBottom: 10 

        }} 

      > 

        <View style={{ color: 'white', fontWeight: 'bold' }}>Thêm sinh viên</View> 

      </View> 

  

      <FlatList 

        data={filteredStudents} 

        renderItem={({ item }) => ( 

          <StudentItem 

            item={item} 

            onEdit={(id) => router.push(`/${id}`)} 

            onDelete={handleDelete} 

          /> 

        )} 

        keyExtractor={(item) => item.id} 

        refreshing={refreshing} 

        onRefresh={handleRefresh} 

      /> 

    </View> 

  ); 

} 

app/add.tsx 

import { useState } from 'react'; 

import { View } from 'react-native'; 

import { useDispatch } from 'react-redux'; 

import { useRouter } from 'expo-router'; 

import { addStudent } from '../store/studentSlice'; 

  

export default function AddStudent() { 

  const router = useRouter(); 

  const dispatch = useDispatch(); 

   

  const [form, setForm] = useState({ 

    name: '', 

    studentId: '', 

    class: '', 

    email: '', 

    phone: '' 

  }); 

  

  const handleSubmit = async () => { 

    if (!form.name.trim() || !form.studentId.trim()) { 

      alert('Vui lòng nhập tên và MSSV'); 

      return; 

    } 

  

    try { 

      await dispatch(addStudent(form) as any).unwrap(); 

      router.back(); 

    } catch (error) { 

      alert('Lỗi khi thêm sinh viên'); 

    } 

  }; 

  

  const updateField = (field: string, value: string) => { 

    setForm(prev => ({ ...prev, [field]: value })); 

  }; 

  

  return ( 

    <View style={{ flex: 1, padding: 10 }}> 

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.name} 

        onChangeText={(text) => updateField('name', text)} 

        placeholder="Tên sinh viên" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.studentId} 

        onChangeText={(text) => updateField('studentId', text)} 

        placeholder="MSSV" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.class} 

        onChangeText={(text) => updateField('class', text)} 

        placeholder="Lớp" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.email} 

        onChangeText={(text) => updateField('email', text)} 

        placeholder="Email" 

        keyboardType="email-address" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 20 

        }} 

        value={form.phone} 

        onChangeText={(text) => updateField('phone', text)} 

        placeholder="Số điện thoại" 

        keyboardType="phone-pad" 

      /> 

  

      <View 

        onPress={handleSubmit} 

        style={{ 

          backgroundColor: '#4CAF50', 

          padding: 12, 

          borderRadius: 5, 

          alignItems: 'center', 

          marginBottom: 10 

        }} 

      > 

        <View style={{ color: 'white', fontWeight: 'bold' }}>Lưu</View> 

      </View> 

  

      <View 

        onPress={() => router.back()} 

        style={{ 

          backgroundColor: '#f44336', 

          padding: 12, 

          borderRadius: 5, 

          alignItems: 'center' 

        }} 

      > 

        <View style={{ color: 'white', fontWeight: 'bold' }}>Hủy</View> 

      </View> 

    </View> 

  ); 

} 

app/[id].tsx 

import { useState, useEffect } from 'react'; 

import { View } from 'react-native'; 

import { useDispatch, useSelector } from 'react-redux'; 

import { useRouter, useLocalSearchParams } from 'expo-router'; 

import { RootState } from '../store'; 

import { updateStudent } from '../store/studentSlice'; 

  

export default function EditStudent() { 

  const { id } = useLocalSearchParams<{ id: string }>(); 

  const router = useRouter(); 

  const dispatch = useDispatch(); 

   

  const student = useSelector((state: RootState) =>  

    state.students.items.find(s => s.id === id) 

  ); 

   

  const [form, setForm] = useState({ 

    id: '', 

    name: '', 

    studentId: '', 

    class: '', 

    email: '', 

    phone: '' 

  }); 

  

  useEffect(() => { 

    if (student) { 

      setForm(student); 

    } 

  }, [student]); 

  

  const handleUpdate = async () => { 

    if (!form.name.trim() || !form.studentId.trim()) { 

      alert('Vui lòng nhập tên và MSSV'); 

      return; 

    } 

  

    try { 

      await dispatch(updateStudent(form) as any).unwrap(); 

      router.back(); 

    } catch (error) { 

      alert('Lỗi khi cập nhật'); 

    } 

  }; 

  

  if (!student) { 

    return ( 

      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}> 

        <View>Không tìm thấy sinh viên</View> 

      </View> 

    ); 

  } 

  

  return ( 

    <View style={{ flex: 1, padding: 10 }}> 

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.name} 

        onChangeText={(text) => setForm({...form, name: text})} 

        placeholder="Tên sinh viên" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.studentId} 

        onChangeText={(text) => setForm({...form, studentId: text})} 

        placeholder="MSSV" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.class} 

        onChangeText={(text) => setForm({...form, class: text})} 

        placeholder="Lớp" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 10 

        }} 

        value={form.email} 

        onChangeText={(text) => setForm({...form, email: text})} 

        placeholder="Email" 

        keyboardType="email-address" 

      /> 

       

      <View 

        style={{ 

          height: 40, 

          borderWidth: 1, 

          borderColor: '#ccc', 

          borderRadius: 5, 

          paddingHorizontal: 10, 

          marginBottom: 20 

        }} 

        value={form.phone} 

        onChangeText={(text) => setForm({...form, phone: text})} 

        placeholder="Số điện thoại" 

        keyboardType="phone-pad" 

      /> 

  

      <View 

        onPress={handleUpdate} 

        style={{ 

          backgroundColor: '#2196F3', 

          padding: 12, 

          borderRadius: 5, 

          alignItems: 'center', 

          marginBottom: 10 

        }} 

      > 

        <View style={{ color: 'white', fontWeight: 'bold' }}>Cập nhật</View> 

      </View> 

  

      <View 

        onPress={() => router.back()} 

        style={{ 

          backgroundColor: '#f44336', 

          padding: 12, 

          borderRadius: 5, 

          alignItems: 'center' 

        }} 

      > 

        <View style={{ color: 'white', fontWeight: 'bold' }}>Hủy</View> 

      </View> 

    </View> 

  ); 

} 

npx create-expo-app student-management --template blank 

cd student-management 

npx expo install @reduxjs/toolkit react-redux axios expo-router 
