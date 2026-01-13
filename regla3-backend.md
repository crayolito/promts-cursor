NESTJS - CLEAN ARCHITECTURE

═══════════════════════════════════════════════════════════════
PRINCIPIOS
═══════════════════════════════════════════════════════════════

1. Todo en ESPAÑOL (carpetas, archivos, variables, comentarios)
2. Dependencias: AFUERA → ADENTRO (nunca al revés)
3. Núcleo PURO: sin frameworks, sin BD, sin librerías externas
4. Puertos definen QUÉ se necesita, Adaptadores definen CÓMO
5. Un caso de uso = Una acción = Un archivo

═══════════════════════════════════════════════════════════════
ESTRUCTURA DE CARPETAS
═══════════════════════════════════════════════════════════════
src/
├── nucleo/ # CAPA INTERNA (pura)
│ ├── entidades/ # Objetos de negocio con reglas
│ │ ├── usuario.entidad.ts
│ │ └── producto.entidad.ts
│ ├── casos-uso/ # Acciones del sistema
│ │ ├── usuario/
│ │ │ ├── crear-usuario.caso-uso.ts
│ │ │ ├── obtener-usuario.caso-uso.ts
│ │ │ └── actualizar-usuario.caso-uso.ts
│ │ └── producto/
│ └── puertos/ # Interfaces/Contratos
│ ├── repositorios/
│ │ └── usuario.repositorio.puerto.ts
│ └── servicios/
│ └── correo.servicio.puerto.ts
│
├── adaptadores/ # CAPA MEDIA (conecta)
│ ├── controladores/
│ │ └── usuario.controlador.ts
│ ├── repositorios/
│ │ └── usuario.repositorio.typeorm.ts
│ └── servicios-externos/
│ └── correo-sendgrid.servicio.ts
│
├── infraestructura/ # CAPA EXTERNA (framework)
│ ├── configuracion/
│ ├── modulos/
│ │ └── usuario.modulo.ts
│ └── base-datos/
│ ├── esquemas/
│ └── migraciones/
│
└── compartido/
├── excepciones/
├── validadores/
└── tipos/

═══════════════════════════════════════════════════════════════
NOMENCLATURA ARCHIVOS
═══════════════════════════════════════════════════════════════
Entidades: [nombre].entidad.ts
Casos de uso: [accion]-[entidad].caso-uso.ts
Puertos: [nombre].[tipo].puerto.ts
Controladores: [nombre].controlador.ts
Repositorios: [nombre].repositorio.[impl].ts
Servicios: [nombre]-[proveedor].servicio.ts
Módulos: [nombre].modulo.ts

═══════════════════════════════════════════════════════════════
REGLAS DE IMPORTACIÓN
═══════════════════════════════════════════════════════════════
PERMITIDO:
✓ Infraestructura → Adaptadores → Núcleo
✓ Casos de Uso → Entidades y Puertos

PROHIBIDO:
✗ Núcleo → Adaptadores o Infraestructura
✗ Entidades → Cualquier otra capa

═══════════════════════════════════════════════════════════════
ENTIDAD (Reglas de negocio puras)
═══════════════════════════════════════════════════════════════
// nucleo/entidades/usuario.entidad.ts

export class Usuario {
private readonly \_id: string;
private \_nombre: string;
private \_correo: string;
private \_activo: boolean;

constructor(props: { id: string; nombre: string; correo: string }) {
// PASO 1: Validamos datos
this.validarNombre(props.nombre);
this.validarCorreo(props.correo);

    // PASO 2: Asignamos valores
    this._id = props.id;
    this._nombre = props.nombre;
    this._correo = props.correo;
    this._activo = true;

}

// ─── Reglas de negocio ───
desactivar(): void {
if (!this.\_activo) throw new Error('Usuario ya desactivado');
this.\_activo = false;
}

// ─── Validaciones privadas ───
private validarCorreo(correo: string): void {
if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(correo)) {
throw new Error('Correo inválido');
}
}

private validarNombre(nombre: string): void {
if (!nombre || nombre.length < 2) {
throw new Error('Nombre debe tener al menos 2 caracteres');
}
}

// ─── Getters ───
get id(): string { return this.\_id; }
get nombre(): string { return this.\_nombre; }
get correo(): string { return this.\_correo; }
get activo(): boolean { return this.\_activo; }
}

═══════════════════════════════════════════════════════════════
PUERTO (Contrato/Interfaz)
═══════════════════════════════════════════════════════════════
// nucleo/puertos/repositorios/usuario.repositorio.puerto.ts

import { Usuario } from '../../entidades/usuario.entidad';

export interface UsuarioRepositorioPuerto {
buscarPorId(id: string): Promise<Usuario | null>;
buscarPorCorreo(correo: string): Promise<Usuario | null>;
guardar(usuario: Usuario): Promise<void>;
actualizar(usuario: Usuario): Promise<void>;
eliminar(id: string): Promise<void>;
existeCorreo(correo: string): Promise<boolean>;
}

═══════════════════════════════════════════════════════════════
CASO DE USO (Una acción del sistema)
═══════════════════════════════════════════════════════════════
// nucleo/casos-uso/usuario/crear-usuario.caso-uso.ts

import { Usuario } from '../../entidades/usuario.entidad';
import { UsuarioRepositorioPuerto } from '../../puertos/repositorios/usuario.repositorio.puerto';
import { CorreoServicioPuerto } from '../../puertos/servicios/correo.servicio.puerto';

export interface CrearUsuarioEntrada {
nombre: string;
correo: string;
}

export interface CrearUsuarioSalida {
id: string;
nombre: string;
correo: string;
}

export class CrearUsuarioCasoUso {
constructor(
private readonly usuarioRepo: UsuarioRepositorioPuerto,
private readonly correoServicio: CorreoServicioPuerto,
) {}

async ejecutar(entrada: CrearUsuarioEntrada): Promise<CrearUsuarioSalida> {
// PASO 1: Verificamos que correo no exista
if (await this.usuarioRepo.existeCorreo(entrada.correo)) {
throw new Error('Correo ya registrado');
}

    // PASO 2: Creamos entidad (valida reglas de negocio)
    const usuario = new Usuario({
      id: crypto.randomUUID(),
      nombre: entrada.nombre,
      correo: entrada.correo,
    });

    // PASO 3: Guardamos en repositorio
    await this.usuarioRepo.guardar(usuario);

    // PASO 4: Enviamos correo de bienvenida
    await this.correoServicio.enviarBienvenida(usuario.correo, usuario.nombre);

    // PASO 5: Retornamos datos
    return {
      id: usuario.id,
      nombre: usuario.nombre,
      correo: usuario.correo,
    };

}
}

═══════════════════════════════════════════════════════════════
CONTROLADOR (Recibe HTTP, llama caso de uso)
═══════════════════════════════════════════════════════════════
// adaptadores/controladores/usuario.controlador.ts

import { Controller, Post, Get, Body, Param, HttpException, HttpStatus } from '@nestjs/common';
import { CrearUsuarioCasoUso } from '../../nucleo/casos-uso/usuario/crear-usuario.caso-uso';

@Controller('usuarios')
export class UsuarioControlador {
constructor(private readonly crearUsuario: CrearUsuarioCasoUso) {}

@Post()
async crear(@Body() datos: { nombre: string; correo: string }) {
try {
// PASO 1: Ejecutamos caso de uso
const resultado = await this.crearUsuario.ejecutar(datos);

      // PASO 2: Retornamos respuesta
      return { exito: true, datos: resultado };
    } catch (error) {
      throw new HttpException(
        { exito: false, mensaje: error.message },
        HttpStatus.BAD_REQUEST,
      );
    }

}
}

═══════════════════════════════════════════════════════════════
REPOSITORIO (Implementa puerto)
═══════════════════════════════════════════════════════════════
// adaptadores/repositorios/usuario.repositorio.typeorm.ts

import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Usuario } from '../../nucleo/entidades/usuario.entidad';
import { UsuarioRepositorioPuerto } from '../../nucleo/puertos/repositorios/usuario.repositorio.puerto';
import { UsuarioModelo } from '../../infraestructura/base-datos/esquemas/usuario.modelo';

@Injectable()
export class UsuarioRepositorioTypeORM implements UsuarioRepositorioPuerto {
constructor(
@InjectRepository(UsuarioModelo)
private readonly repo: Repository<UsuarioModelo>,
) {}

async buscarPorId(id: string): Promise<Usuario | null> {
const modelo = await this.repo.findOne({ where: { id } });
return modelo ? this.aEntidad(modelo) : null;
}

async guardar(usuario: Usuario): Promise<void> {
await this.repo.save(this.aModelo(usuario));
}

async existeCorreo(correo: string): Promise<boolean> {
return (await this.repo.count({ where: { correo } })) > 0;
}

// ─── Conversiones ───
private aEntidad(m: UsuarioModelo): Usuario {
return new Usuario({ id: m.id, nombre: m.nombre, correo: m.correo });
}

private aModelo(e: Usuario): UsuarioModelo {
const m = new UsuarioModelo();
m.id = e.id;
m.nombre = e.nombre;
m.correo = e.correo;
return m;
}
}

═══════════════════════════════════════════════════════════════
MÓDULO (Conecta todo con inyección)
═══════════════════════════════════════════════════════════════
// infraestructura/modulos/usuario.modulo.ts

import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsuarioControlador } from '../../adaptadores/controladores/usuario.controlador';
import { CrearUsuarioCasoUso } from '../../nucleo/casos-uso/usuario/crear-usuario.caso-uso';
import { UsuarioRepositorioTypeORM } from '../../adaptadores/repositorios/usuario.repositorio.typeorm';
import { CorreoSendgridServicio } from '../../adaptadores/servicios-externos/correo-sendgrid.servicio';
import { UsuarioModelo } from '../base-datos/esquemas/usuario.modelo';

@Module({
imports: [TypeOrmModule.forFeature([UsuarioModelo])],
controllers: [UsuarioControlador],
providers: [
// Repositorios
{ provide: 'UsuarioRepositorioPuerto', useClass: UsuarioRepositorioTypeORM },

    // Servicios externos
    { provide: 'CorreoServicioPuerto', useClass: CorreoSendgridServicio },

    // Casos de uso
    {
      provide: CrearUsuarioCasoUso,
      useFactory: (repo, correo) => new CrearUsuarioCasoUso(repo, correo),
      inject: ['UsuarioRepositorioPuerto', 'CorreoServicioPuerto'],
    },

],
})
export class UsuarioModulo {}

═══════════════════════════════════════════════════════════════
RESUMEN RESPONSABILIDADES
═══════════════════════════════════════════════════════════════
┌──────────────────┬────────────────────────────────────────┐
│ CAPA │ RESPONSABILIDAD │
├──────────────────┼────────────────────────────────────────┤
│ Entidades │ Reglas de negocio, validaciones │
│ Casos de Uso │ Orquestan acciones, usan puertos │
│ Puertos │ Contratos (QUÉ, no CÓMO) │
│ Controladores │ Reciben HTTP, llaman casos de uso │
│ Repositorios │ Implementan acceso a datos │
│ Módulos │ Conectan todo con inyección │
└──────────────────┴────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════
COMENTARIOS EN CÓDIGO
═══════════════════════════════════════════════════════════════
Usar PASOS para explicar flujos:
// PASO 1: Validamos entrada
// PASO 2: Ejecutamos lógica
// PASO 3: Guardamos resultado

NO saturar de comentarios. Solo lo necesario.
