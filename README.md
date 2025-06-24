// Fixeo Web App 🛠️ + Chat + Roles + Filtros + Conexión a Firebase
// Framework: React + Firebase Firestore + RealtimeDB + Tailwind CSS

// === components/RegisterForm.jsx (guardando en Firestore) ===
import { useState } from 'react';
import { db } from '../firebase';
import { collection, addDoc } from 'firebase/firestore';

export default function RegisterForm() {
  const [form, setForm] = useState({ nombre: '', servicio: '', ciudad: '', rol: 'cliente' });
  const handleChange = (e) => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await addDoc(collection(db, 'usuarios'), form);
      alert("Registro exitoso en Fixeo");
      setForm({ nombre: '', servicio: '', ciudad: '', rol: 'cliente' });
    } catch (err) {
      console.error("Error al registrar:", err);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="flex flex-col gap-4 mt-4">
      <input type="text" name="nombre" placeholder="Nombre" value={form.nombre} onChange={handleChange} className="p-2 border rounded" required />
      <input type="text" name="servicio" placeholder="Servicio ofrecido" value={form.servicio} onChange={handleChange} className="p-2 border rounded" required />
      <input type="text" name="ciudad" placeholder="Ciudad" value={form.ciudad} onChange={handleChange} className="p-2 border rounded" required />

      <select name="rol" value={form.rol} onChange={handleChange} className="p-2 border rounded">
        <option value="cliente">Cliente</option>
        <option value="profesional">Profesional</option>
      </select>

      <button type="submit" className="bg-teal-600 text-white p-2 rounded">Registrarse en Fixeo</button>
    </form>
  );
}

// === components/ServiceList.jsx (leer Firestore y filtrar) ===
import { useEffect, useState } from 'react';
import { collection, getDocs } from 'firebase/firestore';
import { db } from '../firebase';

const allServices = ["Plomería", "Electricidad", "Diseño", "Fotografía"];

export default function ServiceList() {
  const [profesionales, setProfesionales] = useState([]);
  const [ciudad, setCiudad] = useState('');
  const [servicio, setServicio] = useState('');

  useEffect(() => {
    const cargarUsuarios = async () => {
      const querySnapshot = await getDocs(collection(db, 'usuarios'));
      const data = querySnapshot.docs.map(doc => doc.data());
      const soloProfesionales = data.filter(u => u.rol === 'profesional');
      setProfesionales(soloProfesionales);
    };
    cargarUsuarios();
  }, []);

  const filtrados = profesionales.filter(p =>
    (!ciudad || p.ciudad?.toLowerCase().includes(ciudad.toLowerCase())) &&
    (!servicio || p.servicio === servicio)
  );

  return (
    <div className="mt-4">
      <div className="flex gap-2 mb-4">
        <input
          type="text"
          placeholder="Buscar por ciudad"
          onChange={(e) => setCiudad(e.target.value)}
          className="p-2 border rounded"
        />
        <select
          onChange={(e) => setServicio(e.target.value)}
          className="p-2 border rounded"
        >
          <option value="">Todos los servicios</option>
          {allServices.map((s, i) => (
            <option key={i} value={s}>{s}</option>
          ))}
        </select>
      </div>

      <div className="grid gap-2">
        {filtrados.map((p, i) => (
          <div key={i} className="border p-2 rounded bg-gray-50">
            <strong>{p.nombre}</strong> - {p.servicio} ({p.ciudad})
          </div>
        ))}
        {filtrados.length === 0 && <p>No se encontraron profesionales.</p>}
      </div>
    </div>
  );
}
